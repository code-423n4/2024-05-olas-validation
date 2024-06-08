
### [LOW-0] [DoS] Too many agentIds would cause the initialize function to crash due to MemoryLimitOOG and also gas consumption can be improved

**Description:** 
The variable `_stakingParams` is a variable stored in memory and an array's length is stored in memory as well. Hence, when we have an array.length we will read from memory to find out the length of the variable, which inside a for loop uses too much gas. Having too many agentIds would mean that we will call way too many times the .length of the array from memory which would become more expensive.

**Impact:**
Low, too many agentIds causing the function to be more expensive expensive and also crash due to a MemoryLimitOOG if a larger number of agentIds is inputted. 

## Proof of Concept

```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.25;

import {Test, console} from "forge-std/Test.sol";
import {StdCheats} from "forge-std/StdCheats.sol";
import {StakingToken} from "../contracts/staking/StakingToken.sol";
import {StakingBase} from "../contracts/staking/StakingBase.sol";
import {StakingActivityChecker} from "../contracts/staking/StakingActivityChecker.sol";

 contract StakingTokenTest is StdCheats, Test {

    address public DEPLOYER = makeAddr("deployer");

    uint256 public constant INITIAL_DEPLOYER_BALANCE = 0.01 ether;

    StakingToken public stakingToken;
    StakingActivityChecker public stakingActivityChecker;

    StakingBase.StakingParams public params;

    function setUp() public {

        stakingToken = new StakingToken();
        stakingActivityChecker = new StakingActivityChecker(0x1);

        vm.deal(DEPLOYER, INITIAL_DEPLOYER_BALANCE);

        uint256[] memory agentIdsFirstCall = agentIdsInitialization(1e7);

        params = StakingBase.StakingParams({
            metadataHash: keccak256("test_metadata_hash"),
            maxNumServices: 1e2,
            rewardsPerSecond: 2,
            minStakingDeposit: 2,
            minNumStakingPeriods: 15,
            maxNumInactivityPeriods: 15,
            livenessPeriod: 60,
            timeForEmissions: 36e2,
            numAgentInstances: 5,
            agentIds: agentIdsFirstCall,
            threshold: 3,
            configHash: keccak256("test_config_hash"),
            proxyHash: keccak256("test_proxy_hash"),
            serviceRegistry: address(0x1234),
            activityChecker: address(stakingActivityChecker)
        });

    }

    function testSelfDosAttack() public {
        vm.prank(DEPLOYER);
        stakingToken.initialize(params, address(0x4321), address(0x9876)); 
        // Error MemoryLimitOOG
        // gas: >28717912277

        // 100 agentIds = gas 2971559 (current)
        // 100 agentIds = gas 2970364 (improved)
        // percetange difference = %0.04
    }

    function agentIdsInitialization(uint256 length) public pure returns (uint256[] memory agentIds) {
        agentIds = new uint256[](length);
        uint256 count = 1;
        for (uint256 i = 0; i < length; i++) {
            agentIds[i] = count;
            count++;
        }
    }

}
```

## Tools Used
Solidity, Foundry

**Recommended Mitigation:**

In `StakingBase::_initialize` line 332 add a variable in the following way:

```
+   uint256 agentIdsLength = _stakingParams.agentIds.length;    
-    for (uint256 i = 0; i < _stakingParams.agentIds.length; ++i) {
+    for (uint256 i = 0; i < agentIdsLength; ++i) {
            ...
    }

```

Additionally, consider changing the type of agentIds from uint256 to uint16 or uint32

**--------------------------------**

### [LOW-1] The function_addNominee has internal access and all the input validation is in the functions calling it

**Description:** 

In `governance/contracts/VoteWeighting.sol` we have an internal function `VoteWeighting::_addNominee` which is being called from two other functions `VoteWeighting::addNomineeEVM` and `VoteWeighting::addNomineeNonEVM` which are both external. In both of those external functions we have a input validation, for instance in `VoteWeighting::addNomineeEVM` the input validation is the following:

```
        if (account == address(0)) {
            revert ZeroAddress();
        }

        // Check for zero chain Id
        if (chainId == 0) {
            revert ZeroValue();
        }

        // Check for the chain Id overflow
        if (chainId > MAX_EVM_CHAIN_ID) {
            revert Overflow(chainId, MAX_EVM_CHAIN_ID);
        }
```

However, since the `VoteWeighting::_addNominee` is internal, it can be called from a contract which inherits from `VoteWeighting`, hence bypassing the input validation and the attacker can add nominees which chainId = 0, account address = 0, or a chainId > MAX_EVM_CHAIN_ID.

**Impact:**
LOW

## Proof of Concept

forge test --match-path test/VoteWeightingTest.t.sol

```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.25;

import {Test, console} from "forge-std/Test.sol";
import {StdCheats} from "forge-std/StdCheats.sol";
import {VoteWeighting, Nominee} from "../contracts/VoteWeighting.sol";

contract VoteWeightingTest is StdCheats, Test {

    address public DEPLOYER = makeAddr("deployer");

    uint256 public constant INITIAL_DEPLOYER_BALANCE = 0.01 ether;

    VoteWeighting public voteWeighting;


    Attack public attack;

    function setUp() public {

        vm.deal(DEPLOYER, INITIAL_DEPLOYER_BALANCE);

        voteWeighting = new VoteWeighting(address(0x1234));

        attack = new Attack(address(voteWeighting));
    }

    function testAttack() public {

        bytes32 newAccount = bytes32(uint256(uint160(address(0))));
        uint256 chainId = 0x0;
        uint256 nomineeId = 0x1;

        vm.expectEmit(true, true, true, true);
        emit VoteWeighting.AddNominee(newAccount, chainId, nomineeId);

        attack.execute(newAccount, chainId);
    }

}

contract Attack is VoteWeighting {

    constructor(address _ve) VoteWeighting(_ve) {

    }

    function execute(bytes32 newAccount, uint256 newChainId) public {
    
        Nominee memory newNominee = Nominee({
            account: newAccount,
            chainId: newChainId
        });

        _addNominee(newNominee);
    }

}

```

## Tools Used
Solidity, Foundry

**Recommended Mitigation:**

Make `VoteWeighting::_addNominee` private or add the same input validation in `VoteWeighting::_addNominee` as there is in both `VoteWeighting::addNomineeNonEVM` & `VoteWeighting::addNomineeEVM`
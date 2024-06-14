## L-1: Missing zero address validation


Detect missing zero address validation. 
Check that the address is not zero.


- Found in tokenomics/contracts/Tokenomics.sol [Line: 459](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/Tokenomics.sol#L459)

	```solidity
	    donatorBlacklist = _donatorBlacklist;
	```

- Found in tokenomics/contracts/Tokenomics.sol [Line: 622](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/Tokenomics.sol#L622)

	```solidity
	    donatorBlacklist = _donatorBlacklist;
	```

- Found in tokenomics/contracts/staking/ArbitrumTargetDispenserL2.sol [Line: 48](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/staking/ArbitrumTargetDispenserL2.sol#L48)

	```solidity
	    l1AliasedDepositProcessor = address(uint160(_l1DepositProcessor) + offset);
	```

- Found in registries/contracts/staking/StakingFactory.sol [Line: 105](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/registries/contracts/staking/StakingFactory.sol#L105)

	```solidity
	    verifier = _verifier;
	```

- Found in registries/contracts/staking/StakingFactory.sol [Line: 133](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/registries/contracts/staking/StakingFactory.sol#L133)

	```solidity
	    verifier = _verifier;
	```

- Found in governance/contracts/staking/VoteWeighting.sol [Line: 392](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/governance/contracts/VoteWeighting.sol#L392)

	```solidity
	    dispenser = newDispenser;
	```


## L-2: Missing zero address validation


Detect missing inheritance.
Need to inherit from the missing interface or contract.

Exploit Scenario:
```
    interface ISomething {
        function f1() external returns(uint);
    }

    contract Something {
        function f1() external returns(uint){
            return 42;
        }
    }
```
`Something` should inherit from `ISomething`.


ServiceRegistry [Line: 26-909](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/registries/contracts/ServiceRegistry.sol#L26-L909) should inherit from IService [Line: 30-85](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/registries/contracts/staking/StakingBase.sol#L30-L85)

ServiceRegistryL2 [Line: 27-880](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/registries/contracts/ServiceRegistryL2.sol#L27-L880) should inherit from IService [Line: 30-85](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/registries/contracts/staking/StakingBase.sol#L30-L85)

StakingActivityChecker [Line: 18-63](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/registries/contracts/staking/StakingActivityChecker.sol#L18-L63) should inherit from IActivityChecker [Line: 7-27](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/registries/contracts/staking/StakingBase.sol#L7-L27)

StakingVerifier [Line: 43-259](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/registries/contracts/staking/StakingVerifier.sol#L43-L259) should inherit from IStakingVerifier [Line: 14-29](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/registries/contracts/staking/StakingFactory.sol#L14-L29)

Dispenser [Line: 253-1251](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/Dispenser.sol#L253-L1251) should inherit from IDispenser [Line: 6-8](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/staking/DefaultDepositProcessorL1.sol#L6-L8)



## NC-1: Different pragma directives are used


Detect whether different Solidity versions are used.
Use one Solidity version.

13 different versions of Solidity are used:
	- Version constraint ^0.8.25 is used by:
 		- tokenomics/contracts/Dispenser.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/Dispenser.sol#L2)
		- tokenomics/contracts/Tokenomics.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/Tokenomics.sol#L2)
		- tokenomics/contracts/TokenomicsConstants.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/TokenomicsConstants.sol#L2)
		- tokenomics/contracts/interfaces/IBridgeErrors.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/interfaces/IBridgeErrors.sol#L2)
		- tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/staking/ArbitrumDepositProcessorL1.sol#L2)
		- tokenomics/contracts/staking/ArbitrumTargetDispenserL2.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/staking/ArbitrumTargetDispenserL2.sol#L2)
		- tokenomics/contracts/staking/DefaultDepositProcessorL1.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/staking/DefaultDepositProcessorL1.sol#L2)
		- tokenomics/contracts/staking/DefaultTargetDispenserL2.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol#L2)
		- tokenomics/contracts/staking/EthereumDepositProcessor.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/staking/EthereumDepositProcessor.sol#L2)
		- tokenomics/contracts/staking/GnosisDepositProcessorL1.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/staking/GnosisDepositProcessorL1.sol#L2)
		- tokenomics/contracts/staking/GnosisTargetDispenserL2.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/staking/GnosisTargetDispenserL2.sol#L2)
		- tokenomics/contracts/staking/OptimismDepositProcessorL1.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/staking/OptimismDepositProcessorL1.sol#L2)
		- tokenomics/contracts/staking/OptimismTargetDispenserL2.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/staking/OptimismTargetDispenserL2.sol#L2)
		- tokenomics/contracts/staking/PolygonDepositProcessorL1.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/staking/PolygonDepositProcessorL1.sol#L2)
		- tokenomics/contracts/staking/PolygonTargetDispenserL2.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/staking/PolygonTargetDispenserL2.sol#L2)
		- tokenomics/contracts/staking/WormholeDepositProcessorL1.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/staking/WormholeDepositProcessorL1.sol#L2)
		- tokenomics/contracts/staking/WormholeTargetDispenserL2.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/staking/WormholeTargetDispenserL2.sol#L2)
	- Version constraint ^0.8.18 is used by:
		- tokenomics/contracts/interfaces/IDonatorBlacklist.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/interfaces/IDonatorBlacklist.sol#L2)
		- tokenomics/contracts/interfaces/IErrorsTokenomics.sol [Line: 2](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/interfaces/IErrorsTokenomics.sol#L2)


## NC-2: Missing zero address validation


Detects functions with high (> 11) cyclomatic complexity.
Reduce if possible of cyclomatic complexity by splitting the function into several smaller subroutines.


Dispenser.claimStakingIncentives(uint256,uint256,bytes32,bytes) [Line: 953-1047](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/Dispenser.sol#L953-L1047) has a high cyclomatic complexity (12).

Tokenomics._trackServiceDonations(address,uint256[],uint256[],uint256) [Line: 884-976](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/Dispenser.sol#L884-L976) has a high cyclomatic complexity (15).

Tokenomics.checkpoint() [Line: 1080-1297](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/Dispenser.sol#L1080-L1297) has a high cyclomatic complexity (15).

StakingBase.stake(uint256) [Line: 719-800](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/registries/contracts/staking/StakingBase.sol#L719-L800) has a high cyclomatic complexity (13).


## NC-3: Unused Imports


Importing a file that is not used in the contract likely indicates a mistake. The import should be removed until it is needed.
Remove the unused import. If the import is needed later, it can be added back.


The following unused import(s) in contracts/Tokenomics.sol should be removed: 
	-import {convert, UD60x18} from "@prb/math/src/UD60x18.sol"; [Line: 4](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/Tokenomics.sol#L4)
	-import {IErrorsTokenomics} from "./interfaces/IErrorsTokenomics.sol"; [Line: 7](https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/Tokenomics.sol#L7)

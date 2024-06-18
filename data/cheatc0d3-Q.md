# QA Report: Olas

## L-01: No Timelocks when changing critical addresses

### Impact
The `changeDispenser` function in the VoteWeighting contract allows the contract owner to unilaterally change the address of the dispenser contract without any consensus or approval mechanism.

The contract owner can potentially change the dispenser contract to a malicious or untrusted contract, which could compromise the integrity and reliability of nominee management in the VoteWeighting contract.

Additionally, if the owner's private key is compromised, an attacker could exploit this centralization issue to change the dispenser contract to a malicious one, potentially leading to further attacks or unauthorized actions.

### Proof of Concept

```solidity
function changeDispenser(address newDispenser) external {
    // Check for the contract ownership
    if (msg.sender != owner) {
        revert OwnerOnly(msg.sender, owner);
    }

    dispenser = newDispenser;
    emit DispenserUpdated(newDispenser);
}
```

Only the contract `owner` can call the `changeDispenser` function and update the `dispenser` state variable to a new address (`newDispenser`). There are no additional checks, approvals, or time-locks in place to prevent the owner from making this change unilaterally.

### Recommended Mitigation Steps
Implement a time-lock mechanism that requires a certain period of time to pass before the dispenser contract change takes effect. Here's an example of how you can modify the `changeDispenser` function to include a time-lock:

```solidity
uint256 public constant TIMELOCK_DELAY = 3 days; // Adjust the delay as needed
uint256 public dispenserTimeLock; // Timestamp when the dispenser change will take effect

function changeDispenser(address newDispenser) external {
    // Check for the contract ownership
    if (msg.sender != owner) {
        revert OwnerOnly(msg.sender, owner);
    }

    // Set the time-lock for the dispenser change
    dispenserTimeLock = block.timestamp + TIMELOCK_DELAY;

    // Emit an event to notify stakeholders about the pending change
    emit DispenserChangeProposed(newDispenser, dispenserTimeLock);
}

function executeDispenserChange() external {
    // Check if the time-lock period has passed
    if (block.timestamp < dispenserTimeLock) {
        revert TimeLockNotExpired(dispenserTimeLock);
    }

    dispenser = newDispenser;
    emit DispenserUpdated(newDispenser);
}
```

With this modification, when the owner calls the `changeDispenser` function, instead of immediately updating the dispenser contract address, it sets a time-lock (`dispenserTimeLock`) with a delay of `TIMELOCK_DELAY` (e.g., 3 days). The `DispenserChangeProposed` event is emitted to notify stakeholders about the pending change and the time when it will take effect.

To execute the actual dispenser change, the `executeDispenserChange` function must be called after the time-lock period has expired. This gives stakeholders time to review and potentially object to the proposed change.

## L-02: abi.encode is safer than abi.encodePacked

### Impact
Using `abi.encode` instead of `abi.encodePacked` would provide an additional layer of safety by including type information in the encoding.

### Proof of Concept
The `abi.encodePacked` function is used in the following lines of code:

```solidity
bytes32 salt = keccak256(abi.encodePacked(block.chainid, localNonce));
```

```solidity
bytes memory deploymentData = abi.encodePacked(
    type(StakingProxy).creationCode,
    uint256(uint160(implementation))
);
```

```solidity
bytes32 hash = keccak256(
    abi.encodePacked(
        bytes1(0xff), address(this), salt, keccak256(deploymentData)
    )
);
```

While these usages are generally safe due to the fixed sizes of the types being packed, using `abi.encode` would include additional type information, making the encoding even more robust.

### Tools Used
Manual code review

### Recommended Mitigation Steps
Consider replacing `abi.encodePacked` with `abi.encode` in the identified lines of code. This will include type information in the encoding, providing an extra layer of safety. 

## L-03: Incorrect Encoding of multisig Parameter in abi.encodeCall Leads to Potential Reverts and Incorrect Behavior

### Impact

Passing the `multisig` parameter directly to `abi.encodeCall` instead of as a tuple when calling the `getMultisigNonces` function on the `activityChecker` contract will likely result in an incorrect encoding of the function call data. This can lead to the `activityChecker` contract receiving invalid input and potentially reverting or returning unexpected results. As a consequence, the `_checkRatioPass` function may not correctly determine if the service has passed the activity check, affecting the overall behavior of the staking contract.

### Proof of Concept

https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/registries/contracts/staking/StakingBase.sol#L406

```solidity
bytes memory activityData = abi.encodeCall(IActivityChecker.getMultisigNonces, multisig);
```

It should be updated to:

```solidity
bytes memory activityData = abi.encodeCall(IActivityChecker.getMultisigNonces, (multisig));
```

The `abi.encodeCall` function expects the parameters to be passed as a tuple, even if there is only one parameter. Failing to wrap the `multisig` parameter in a tuple will result in an incorrect encoding of the function call data.

## L-04: Array Length Mismatch Vulnerability

### Impact
The lack of array length validation in the `sendMessageBatch` function can lead to incorrect allocation of staking incentives, potential reentrancy vulnerabilities, and general instability in the contract's operations, possibly resulting in the loss of funds or service disruption.

### Proof of Concept
The `sendMessageBatch` function does not check that `targets` and `stakingIncentives` arrays have the same length:

```solidity
function sendMessageBatch(
    address[] memory targets,
    uint256[] memory stakingIncentives,
    bytes memory bridgePayload,
    uint256 transferAmount
) external virtual payable {
    // Check for the dispenser contract to be the msg.sender
    if (msg.sender != l1Dispenser) {
        revert ManagerOnly(l1Dispenser, msg.sender);
    }

    // Send the message to L2
    uint256 sequence = _sendMessage(targets, stakingIncentives, bridgePayload, transferAmount);

    // Increase the staking batch nonce
    stakingBatchNonce++;

    emit MessagePosted(sequence, targets, stakingIncentives, transferAmount);
}
```

### Recommended Mitigation Steps
Add an array length check to ensure `targets` and `stakingIncentives` have the same length:

```solidity
error ArrayLengthMismatch();

function sendMessageBatch(
    address[] memory targets,
    uint256[] memory stakingIncentives,
    bytes memory bridgePayload,
    uint256 transferAmount
) external virtual payable {
    // Check for the dispenser contract to be the msg.sender
    if (msg.sender != l1Dispenser) {
        revert ManagerOnly(l1Dispenser, msg.sender);
    }

    // Ensure the targets and stakingIncentives arrays have the same length
    if (targets.length != stakingIncentives.length) {
        revert ArrayLengthMismatch();
    }

    // Send the message to L2
    uint256 sequence = _sendMessage(targets, stakingIncentives, bridgePayload, transferAmount);

    // Increase the staking batch nonce
    stakingBatchNonce++;

    emit MessagePosted(sequence, targets, stakingIncentives, transferAmount);
}
```

## L-05: Insufficient Balance Checks for OLAS Tokens in `_sendMessage` Function

### Impact
The `_sendMessage` function in the `OptimismDepositProcessorL1` contract assumes that the contract always has enough OLAS tokens to cover `transferAmount`. This can cause the transaction to revert if the balance is insufficient.

A transaction will fail if the contract does not have enough OLAS tokens, disrupting the intended token transfers to L2.

### Proof of Concept

https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/staking/OptimismDepositProcessorL1.sol#L114

```solidity
if (transferAmount > 0) {
    IToken(olas).approve(l1TokenRelayer, transferAmount);
    IBridge(l1TokenRelayer).depositERC20To(olas, olasL2, l2TargetDispenser, transferAmount, uint32(TOKEN_GAS_LIMIT), "");
}
```

The code does not check the balance before approving and transferring tokens. If `transferAmount` exceeds the balance, the transaction reverts.

### Recommended Mitigation Steps
Add a balance check before the token transfer:

```solidity
if (transferAmount > 0) {
    uint256 balance = IToken(olas).balanceOf(address(this));
    require(balance >= transferAmount, "Insufficient OLAS balance");

    IToken(olas).approve(l1TokenRelayer, transferAmount);
    IBridge(l1TokenRelayer).depositERC20To(olas, olasL2, l2TargetDispenser, transferAmount, uint32(TOKEN_GAS_LIMIT), "");
}
```

## L-06: Lack of receive Function in Proxy Contract Prevents Handling of Plain Ether Transfers

### Impact
The contract does not define a `receive` function, which means that the proxy contract cannot handle plain Ether transfers without data. This limitation can result in failed Ether transfers when users send plain Ether directly to the proxy contract without any accompanying data. This restricts the usability of the proxy contract in scenarios where it needs to receive plain Ether.

### Poc
https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/registries/contracts/staking/StakingProxy.sol#L40

```solidity
fallback() external payable {
    assembly {
        let implementation := sload(SERVICE_STAKING_PROXY)
        calldatacopy(0, 0, calldatasize())
        let success := delegatecall(gasleft(), implementation, 0, calldatasize(), 0, 0)
        returndatacopy(0, 0, returndatasize())
        if eq(success, 0) {
            revert(0, returndatasize())
        }
        return(0, returndatasize())
    }
}
```

### Recommended Mitigation Steps
Implement a `receive` function to handle plain Ether transfers. The `receive` function can either forward the Ether to the implementation contract or revert if Ether transfers are not allowed.


## L-07: Initialization of inflationPerSecond Fails if getInflationForYear(0) Returns Zero, Leading to Contract Deployment Failure

### Impact
The `initializeTokenomics` function assumes that `getInflationForYear(0)` always returns a non-zero value without proper validation. If `getInflationForYear(0)` returns zero, it will lead to a division by zero error when calculating `_inflationPerSecond`. This can cause the contract initialization to fail or revert, preventing the contract from being deployed and used correctly.

### Proof of Concept

https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol#L473

```solidity
uint256 _inflationPerSecond = getInflationForYear(0) / zeroYearSecondsLeft;
```

If `getInflationForYear(0)` returns zero, the division by `zeroYearSecondsLeft` will result in a division by zero error.

### Recommended Mitigation Steps
Add validation to ensure that `getInflationForYear(0)` returns a non-zero value before performing the division.

```solidity
uint256 initialInflation = getInflationForYear(0);
require(initialInflation > 0, "Initial inflation must be greater than 0");
uint256 _inflationPerSecond = initialInflation / zeroYearSecondsLeft;
```

## L-08: Unvalidated Implementation Address Retrieval in tokenomicsImplementation Allows Execution of Malicious Code

### Impact
The `tokenomicsImplementation` function retrieves the implementation address using inline assembly without proper validation. If the `PROXY_TOKENOMICS` storage slot does not contain a valid address or contains an address that is not a contract, it can lead to unexpected behavior or security vulnerabilities. An attacker might be able to manipulate the storage slot to point to a malicious contract, allowing them to execute arbitrary code or manipulate the contract's state.

### Proof of Concept

https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol#L518

```solidity
assembly {
    implementation := sload(PROXY_TOKENOMICS)
}
```

### Recommended Mitigation Steps
Add validation to ensure that the retrieved `implementation` address is a valid contract address.

```solidity
assembly {
    implementation := sload(PROXY_TOKENOMICS)
}
require(implementation != address(0), "Implementation address is not set");
require(implementation.code.length > 0, "Implementation address must be a contract");
```

If either of these checks fails, the function will revert with an appropriate error message.

## L-09: Lack of Validation in changeTokenomicsImplementation Allows Setting Invalid or Malicious Implementation Address

### Impact
The `changeTokenomicsImplementation` function allows the contract owner to change the implementation address of the tokenomics contract. However, it lacks a mechanism to ensure that the new implementation address is a valid and authorized contract. If the owner accidentally or maliciously sets the implementation address to an unauthorized, non-contract, or malicious contract address, it can lead to unexpected behavior, loss of functionality, or even loss of funds. Users interacting with the contract may unknowingly use an incorrect or malicious implementation, exposing them to risks and vulnerabilities.

### Proof of Concept

https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol#L539

```solidity
assembly {
    sstore(PROXY_TOKENOMICS, implementation)
}
emit TokenomicsImplementationUpdated(implementation);
```


### Recommended Mitigation Steps
Implement a whitelist mechanism to restrict the allowed implementation addresses. Only whitelisted addresses should be permitted to be set as the new implementation address. 

```solidity
// Whitelist of authorized implementation addresses
mapping(address => bool) public authorizedImplementations;

function changeTokenomicsImplementation(address implementation) external {
    require(msg.sender == owner, "Only the contract owner can change the implementation");
    require(authorizedImplementations[implementation], "Implementation address is not authorized");

    assembly {
        sstore(PROXY_TOKENOMICS, implementation)
    }
    emit TokenomicsImplementationUpdated(implementation);
}

function addAuthorizedImplementation(address implementation) external {
    require(msg.sender == owner, "Only the contract owner can authorize implementations");
    require(implementation.code.length > 0, "Implementation address must be a contract");
    authorizedImplementations[implementation] = true;
    emit ImplementationAuthorized(implementation);
}

function removeAuthorizedImplementation(address implementation) external {
    require(msg.sender == owner, "Only the contract owner can remove authorized implementations");
    authorizedImplementations[implementation] = false;
    emit ImplementationUnauthorized(implementation);
}
```


## L-10: Missing Event Emission of Previous Owner Address in changeOwner Reduces Transparency

### Impact
The `changeOwner` function allows the current owner to transfer ownership of the contract to a new owner. However, it lacks an event emission to track the previous owner. This absence of event logging can lead to reduced transparency and accountability in the ownership transfer process. In case of disputes or audits, it may be difficult to trace the history of ownership changes without a proper event trail. Moreover, the lack of an event emission may make it challenging for off-chain tools and interfaces to accurately reflect the current ownership state of the contract.

### Proof of Concept
The affected lines of code are:
```solidity
function changeOwner(address newOwner) external {
    if (msg.sender != owner) {
        revert OwnerOnly(msg.sender, owner);
    }
    if (newOwner == address(0)) {
        revert ZeroAddress();
    }
    owner = newOwner;
    emit OwnerUpdated(newOwner);
}
```

### Recommended Mitigation Steps
Include the previous owner's address in the event emission.

```solidity
function changeOwner(address newOwner) external {
    if (msg.sender != owner) {
        revert OwnerOnly(msg.sender, owner);
    }
    if (newOwner == address(0)) {
        revert ZeroAddress();
    }
    address previousOwner = owner;
    owner = newOwner;
    emit OwnerUpdated(previousOwner, newOwner);
}
```

## L-11: Non-atomic Update of Manager Addresses in changeManagers Can Lead to Inconsistent State and Incorrect Behavior

### Impact
The `changeManagers` function allows the contract owner to update the addresses of critical manager contracts such as the treasury, depository, and dispenser. However, the current implementation handles each address update independently, which can lead to potential inconsistencies if one or more updates fail while others succeed. This partial update scenario can result in an incorrect or unexpected state of the manager contracts, causing issues with the overall functioning of the system. For example, if the treasury address is updated successfully but the depository or dispenser address update fails, the contract may interact with mismatched or outdated manager contracts, leading to incorrect behavior or loss of funds.

### Proof of Concept

https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol#L565

```solidity
function changeManagers(address _treasury, address _depository, address _dispenser) external {
    if (msg.sender != owner) {
        revert OwnerOnly(msg.sender, owner);
    }
    if (_treasury != address(0)) {
        treasury = _treasury;
        emit TreasuryUpdated(_treasury);
    }
    if (_depository != address(0)) {
        depository = _depository;
        emit DepositoryUpdated(_depository);
    }
    if (_dispenser != address(0)) {
        dispenser = _dispenser;
        emit DispenserUpdated(_dispenser);
    }
}
```

The code updates the `treasury`, `depository`, and `dispenser` addresses separately based on the provided input values. Each update is performed independently, and the corresponding events are emitted. However, if any of the updates fail (e.g., due to a revert or an exception), the previously successful updates will still take effect, resulting in a partial update of the manager contracts.

### Recommended Mitigation Steps
It is recommended to implement a mechanism that handles the updates as a single transaction.

```solidity
function changeManagers(address _treasury, address _depository, address _dispenser) external {
    if (msg.sender != owner) {
        revert OwnerOnly(msg.sender, owner);
    }
    if (_treasury == address(0) || _depository == address(0) || _dispenser == address(0)) {
        revert ZeroAddress();
    }
    treasury = _treasury;
    depository = _depository;
    dispenser = _dispenser;
    emit ManagersUpdated(_treasury, _depository, _dispenser);
}
```

## L-12: Independent Update of Registry Addresses in changeRegistries Can Lead to Inconsistent State

### Impact
The `changeRegistries` function allows the contract owner to update the addresses of the component registry, agent registry, and service registry contracts. However, the current implementation updates each address independently, which can result in inconsistencies if one or more updates fail while others succeed.

This partial update scenario can leave the registry contracts in an incorrect or mismatched state, leading to issues with the overall functionality of the system. For instance, if the component registry address is updated successfully, but the agent or service registry address update fails, the contract may interact with mismatched or outdated registry contracts. This can cause incorrect behavior, loss of functionality, or even security vulnerabilities.

### Proof of Concept

https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol#L592

```solidity
function changeRegistries(address _componentRegistry, address _agentRegistry, address _serviceRegistry) external {
    if (msg.sender != owner) {
        revert OwnerOnly(msg.sender, owner);
    }
    if (_componentRegistry != address(0)) {
        componentRegistry = _componentRegistry;
        emit ComponentRegistryUpdated(_componentRegistry);
    }
    if (_agentRegistry != address(0)) {
        agentRegistry = _agentRegistry;
        emit AgentRegistryUpdated(_agentRegistry);
    }
    if (_serviceRegistry != address(0)) {
        serviceRegistry = _serviceRegistry;
        emit ServiceRegistryUpdated(_serviceRegistry);
    }
}
```

As shown above, the code updates each registry address separately based on the provided input values. Each update is performed independently, and the corresponding events are emitted. However, if any of the updates fail (e.g., due to a revert or an exception), the previously successful updates will still take effect, resulting in a partial update of the registry contracts.

### Recommended Mitigation Steps

Modify the `changeRegistries` function to update all registry addresses atomically within a single transaction. If any of the updates fail, the entire transaction should revert, ensuring that no partial updates occur.


```solidity
function changeRegistries(address _componentRegistry, address _agentRegistry, address _serviceRegistry) external {
    if (msg.sender != owner) {
        revert OwnerOnly(msg.sender, owner);
    }
    if (_componentRegistry == address(0) || _agentRegistry == address(0) || _serviceRegistry == address(0)) {
        revert InvalidAddress();
    }
    componentRegistry = _componentRegistry;
    agentRegistry = _agentRegistry;
    serviceRegistry = _serviceRegistry;
    emit RegistriesUpdated(_componentRegistry, _agentRegistry, _serviceRegistry);
}
```


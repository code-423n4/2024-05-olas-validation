From the `TokenomicsConstants.sol` file, the `getSupplyCapForYear` function calculates the supply cap for a specific year based on the number of years passed from the launch date. There are a few errors in the code comment 
https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/TokenomicsConstants.sol#L28
1. Documentation Comment Correction:
The line 1e27 * (1.02)^(x-9) for x >= 10 should be corrected to 10e27 * (1.02)^(x-9) for x >= 10 to accurately reflect the logic used in the function. The initial supply cap for the 9th year is based on 1_000_000_000e18 which is 10e27.

2. Missing Documentation on Initial Supply Cap:
There is no explicit comment stating that the value at index zero in the supplyCaps array represents the initial supply cap at deployment. This should be added for clarity.

3. Comment Improvement:
The comment // For the first 10 years the supply caps are pre-defined should be enhanced to // For the first 10 years the supply caps are pre-defined, year of deployment inclusive to clearly indicate that the first year includes the deployment year.

# No limit check 
The maximum number of agent is used to specify the max number of agent instance that can be associated with staking service in the contest of the stakingBase.sol smart contract. But there wanst a limit on the number of instances that can be created for a particular service, snesuring that the service doesn not exceed the predefinde capacity what it just does is that it requires that ```numAgentInstance``` == ```numAgentInstance```

### Recommendation
     if (numAgentInstances >= service.maxNumAgentInstances) {
            revert WrongServiceConfiguration(serviceId);
        }


# Reinitializing new size be gas expensive
```
 function _calculateStakingRewards() internal view returns (
        uint256 lastAvailableRewards,
        uint256 numServices,
        uint256 totalRewards,
        uint256[] memory eligibleServiceIds,
        uint256[] memory eligibleServiceRewards,
        uint256[] memory serviceIds,
        uint256[][] memory serviceNonces,
        uint256[] memory serviceInactivity
    )
    {
        // Check the last checkpoint timestamp and the liveness period, also check for available rewards to be not zero
        uint256 tsCheckpointLast = tsCheckpoint;
        lastAvailableRewards = availableRewards;
        if (block.timestamp - tsCheckpointLast >= livenessPeriod && lastAvailableRewards > 0) {
            // Get the service Ids set length
            uint256 size = setServiceIds.length;

            // Get necessary arrays
            serviceIds = new uint256[](size);
            eligibleServiceIds = new uint256[](size);
            eligibleServiceRewards = new uint256[](size);
            serviceNonces = new uint256[][](size);
            serviceInactivity = new uint256[](size);

            // Calculate each staked service reward eligibility
            for (uint256 i = 0; i < size; ++i) {
                // Get current service Id
                serviceIds[i] = setServiceIds[i];

                // Get the service info
                ServiceInfo storage sInfo = mapServiceInfo[serviceIds[i]];
                // For brevity
            }
        }
    }
```

Here, several arrays are initialized based on the number of services (size). These arrays will store information about services, their eligibility for rewards, and other related data.

The arrays serviceIds, eligibleServiceIds, eligibleServiceRewards, serviceNonces, and serviceInactivity are re-initialized inside the if block every time it executes. This could lead to unnecessary gas costs if the arrays are large.

##### Recommendation
To reduce the gas cost, you can remove the reinitialization  from the `if` statment block outside;

```
function _calculateStakingRewards() internal view returns (
    uint256 lastAvailableRewards,
    uint256 numServices,
    uint256 totalRewards,
    uint256[] memory eligibleServiceIds,
    uint256[] memory eligibleServiceRewards,
    uint256[] memory serviceIds,
    uint256[][] memory serviceNonces,
    uint256[] memory serviceInactivity
)
{
    // Initialize necessary arrays
    uint256 size = setServiceIds.length;
    serviceIds = new uint256[](size);
    eligibleServiceIds = new uint256[](size);
    eligibleServiceRewards = new uint256[](size);
    serviceNonces = new uint256[][](size);
    serviceInactivity = new uint256[](size);

    // Check the last checkpoint timestamp and the liveness period, also check for available rewards to be not zero
    uint256 tsCheckpointLast = tsCheckpoint;
    lastAvailableRewards = availableRewards;
    if (block.timestamp - tsCheckpointLast >= livenessPeriod && lastAvailableRewards > 0) {
        // Calculate each staked service reward eligibility
        for (uint256 i = 0; i < size; ++i) {
            // Get current service Id
            serviceIds[i] = setServiceIds[i];

            // Get the service info
            ServiceInfo storage sInfo = mapServiceInfo[serviceIds[i]];
        }
   }
}
```

## [QA] Informational Findings on VoteWeighting.sol
## [QA-1]  TITLE : QA Report for Error Functions
***Description:***
This report examines the error handling functions within the file to ensure they adhere to best practices for code readability, functionality, and naming conventions. Each error function is named according to the format Tokenomics__Name(), which facilitates easy identification and debugging of specific issues related to file operations.
```diff
/// @dev Only `owner` has a privilege, but the `sender` was provided.
/// @param sender Sender address.
/// @param owner Required sender address as an owner.
- error OwnerOnly(address sender, address owner);
+ error VoteWeighting__OwnerOnly(address sender, address owner);

/// @dev Provided zero address.
- error ZeroAddress();
+ error VoteWeighting__ZeroAddress();

/// @dev Zero value when it has to be different from zero.
- error ZeroValue();
+ error VoteWeighting__ZeroValue();

/// @dev Wrong length of two arrays.
/// @param numValues1 Number of values in a first array.
/// @param numValues2 Number of values in a second array.
- error WrongArrayLength(uint256 numValues1, uint256 numValues2);
+ error VoteWeighting__WrongArrayLength(uint256 numValues1, uint256 numValues2);

/// @dev Value overflow.
/// @param provided Overflow value.
/// @param max Maximum possible value.
- error Overflow(uint256 provided, uint256 max);
+ error VoteWeighting__Overflow(uint256 provided, uint256 max);

/// @dev Underflow value.
/// @param provided Provided value.
/// @param expected Minimum expected value.
- error Underflow(uint256 provided, uint256 expected);
+ error VoteWeighting__Underflow(uint256 provided, uint256 expected);

/// @dev Nominee does not exist.
/// @param account Nominee account address.
/// @param chainId Nominee chain Id.
- error NomineeDoesNotExist(bytes32 account, uint256 chainId);
+ error VoteWeighting__NomineeDoesNotExist(bytes32 account, uint256 chainId);

/// @dev Nominee already exists.
/// @param account Nominee account address.
/// @param chainId Nominee chain Id.
- error NomineeAlreadyExists(bytes32 account, uint256 chainId);
+ error VoteWeighting__NomineeAlreadyExists(bytes32 account, uint256 chainId);

/// @dev Value lock is expired.
/// @param account Address that is checked for the locked value.
/// @param deadline The lock expiration deadline.
/// @param curTime Current timestamp.
- error LockExpired(address account, uint256 deadline, uint256 curTime);
+ error VoteWeighting__LockExpired(address account, uint256 deadline, uint256 curTime);

/// @dev Violated a negative slope value.
/// @param account Account address.
/// @param slope Negative slope.
- error NegativeSlope(address account, int128 slope);
+ error VoteWeighting__NegativeSlope(address account, int128 slope);

/// @dev The vote has been performed already.
/// @param voter Voter address.
/// @param curTime Current time.
/// @param nextAllowedVotingTime Next allowed voting time.
- error VoteTooOften(address voter, uint256 curTime, uint256 nextAllowedVotingTime);
+ error VoteWeighting__VoteTooOften(address voter, uint256 curTime, uint256 nextAllowedVotingTime);

/// @dev Nominee is not in the removed nominee map.
/// @param account Nominee account address.
/// @param chainId Nominee chain Id.
- error NomineeNotRemoved(bytes32 account, uint256 chainId);
+ error VoteWeighting__NomineeNotRemoved(bytes32 account, uint256 chainId);

/// @dev Nominee is in the removed nominee map.
/// @param account Nominee account address.
/// @param chainId Nominee chain Id.
- error NomineeRemoved(bytes32 account, uint256 chainId);
+ error VoteWeighting__NomineeRemoved(bytes32 account, uint256 chainId);

```

## [QA] Informational Findings on Tokenomics.sol
## [QA-2]  TITLE : QA Report for Error Functions

***Description:***
This report examines the error handling functions within the file to ensure they adhere to best practices for code readability, functionality, and naming conventions. Each error function is named according to the format Name__error(), which facilitates easy identification and debugging of specific issues related to file operations.
```diff

/// @dev Only `manager` has a privilege, but the `sender` was provided.
/// @param sender Sender address.
/// @param manager Required sender address as a manager.
-  error ManagerOnly(address sender, address manager);
+  error Tokenomics__ManagerOnly(address sender, address manager);


/// @dev Only `owner` has a privilege, but the `sender` was provided.
/// @param sender Sender address.
/// @param owner Required sender address as an owner.
error OwnerOnly(address sender, address owner);

/// @dev Provided zero address.
- error ZeroAddress();
+ error Tokenomics__ZeroAddress();

/// @dev Wrong length of two arrays.
/// @param numValues1 Number of values in a first array.
/// @param numValues2 Number of values in a second array.
- error WrongArrayLength(uint256 numValues1, uint256 numValues2);
+ error Tokenomics__WrongArrayLength(uint256 numValues1, uint256 numValues2);

/// @dev Service Id does not exist in registry records.
/// @param serviceId Service Id.
- error ServiceDoesNotExist(uint256 serviceId);
+ error Tokenomics__ServiceDoesNotExist(uint256 serviceId);


/// @dev Zero value when it has to be different from zero.
- error ZeroValue();
+ error Tokenomics__ZeroValue();
/// @dev Value overflow.
/// @param provided Overflow value.
/// @param max Maximum possible value.
- error Overflow(uint256 provided, uint256 max);
+ error Tokenomics__Overflow(uint256 provided, uint256 max);


/// @dev Service was never deployed.
/// @param serviceId Service Id.
- error ServiceNeverDeployed(uint256 serviceId);
+ error Tokenomics__ServiceNeverDeployed(uint256 serviceId);

/// @dev Received lower value than the expected one.
/// @param provided Provided value is lower.
/// @param expected Expected value.
- error LowerThan(uint256 provided, uint256 expected);
+ error Tokenomics__LowerThan(uint256 provided, uint256 expected);

/// @dev Wrong amount received / provided.
/// @param provided Provided amount.
/// @param expected Expected amount.
- error WrongAmount(uint256 provided, uint256 expected);
+ error Tokenomics__WrongAmount(uint256 provided, uint256 expected);

/// @dev The donator address is blacklisted.
/// @param account Donator account address.
- error DonatorBlacklisted(address account);
+ error Tokenomics__DonatorBlacklisted(address account);

/// @dev The contract is already initialized.
- error AlreadyInitialized();
+ error Tokenomics__AlreadyInitialized();

/// @dev The contract has to be delegate-called via proxy.
- error DelegatecallOnly();
+ error Tokenomics__DelegatecallOnly();

/// @dev Caught an operation that is not supposed to happen in the same block.
- error SameBlockNumberViolation();
+ error Tokenomics__SameBlockNumberViolation();

/// @dev Failure of treasury re-balance during the reward allocation.
/// @param epochNumber Epoch number.
- error TreasuryRebalanceFailed(uint256 epochNumber);
+ error Tokenomics__TreasuryRebalanceFailed(uint256 epochNumber);

/// @dev Operation with a wrong component / agent Id.
/// @param unitId Component / agent Id.
/// @param unitType Type of the unit (component / agent).
- error WrongUnitId(uint256 unitId, uint256 unitType);
+ error Tokenomics__WrongUnitId(uint256 unitId, uint256 unitType);

```

## [QA] Informational Findings on StakingBase.sol
## [QA-3]  TITLE : QA Report for Error Functions
```diff
/// @dev Only `owner` has a privilege, but the `sender` was provided.
/// @param sender Sender address.
/// @param owner Required sender address as an owner.
- error OwnerOnly(address sender, address owner);
+ error StakingBase__OwnerOnly(address sender, address owner);

/// @dev Provided zero address.
- error ZeroAddress();
+ error StakingBase__ZeroAddress();

/// @dev Provided zero value.
- error ZeroValue();
+ error StakingBase__ZeroValue();

/// @dev The deployed activity checker must be a contract.
/// @param activityChecker Activity checker address.
- error ContractOnly(address activityChecker);
+ error StakingBase__ContractOnly(address activityChecker);

/// @dev Agent Id is not correctly provided for the current routine.
/// @param agentId Component Id.
- error WrongAgentId(uint256 agentId);
+ error StakingBase__WrongAgentId(uint256 agentId);

/// @dev Wrong state of a service.
/// @param state Service state.
/// @param serviceId Service Id.
- error WrongServiceState(uint256 state, uint256 serviceId);
+ error StakingBase__WrongServiceState(uint256 state, uint256 serviceId);

/// @dev Multisig is not whitelisted.
/// @param multisig Address of a multisig implementation.
- error UnauthorizedMultisig(address multisig);
+ error StakingBase__UnauthorizedMultisig(address multisig);

/// @dev The contract is already initialized.
- error AlreadyInitialized();
+ error StakingBase__AlreadyInitialized();

/// @dev No rewards are available in the contract.
- error NoRewardsAvailable();
+ error StakingBase__NoRewardsAvailable();

/// @dev Maximum number of staking services is reached.
/// @param maxNumServices Maximum number of staking services.
- error MaxNumServicesReached(uint256 maxNumServices);
+ error StakingBase__MaxNumServicesReached(uint256 maxNumServices);

/// @dev Received lower value than the expected one.
/// @param provided Provided value is lower.
/// @param expected Expected value.
- error LowerThan(uint256 provided, uint256 expected);
+ error StakingBase__LowerThan(uint256 provided, uint256 expected);

/// @dev Required service configuration is wrong.
/// @param serviceId Service Id.
- error WrongServiceConfiguration(uint256 serviceId);
+ error StakingBase__WrongServiceConfiguration(uint256 serviceId);

/// @dev Service is not unstaked.
/// @param serviceId Service Id.
- error ServiceNotUnstaked(uint256 serviceId);
+ error StakingBase__ServiceNotUnstaked(uint256 serviceId);

/// @dev Service is not found.
/// @param serviceId Service Id.
- error ServiceNotFound(uint256 serviceId);
+ error StakingBase__ServiceNotFound(uint256 serviceId);

/// @dev Service was not staked a minimum required time.
/// @param serviceId Service Id.
/// @param tsProvided Time the service is staked for.
/// @param tsExpected Minimum time the service needs to be staked for.
- error NotEnoughTimeStaked(uint256 serviceId, uint256 tsProvided, uint256 tsExpected);
+ error StakingBase__NotEnoughTimeStaked(uint256 serviceId, uint256 tsProvided, uint256 tsExpected);
```


## [QA] Informational Findings on StakingFactory.sol
## [QA-4]  TITLE : QA Report for Error Functions

```diff
/// @dev Only `owner` has a privilege, but the `sender` was provided.
/// @param sender Sender address.
/// @param owner Required sender address as an owner.
- error OwnerOnly(address sender, address owner);
+ error StakingFactory__OwnerOnly(address sender, address owner);

/// @dev Provided zero address.
- error ZeroAddress();
+ error StakingFactory__ZeroAddress();

/// @dev Provided incorrect data length.
/// @param expected Expected minimum data length.
/// @param provided Provided data length.
- error IncorrectDataLength(uint256 expected, uint256 provided);
+ error StakingFactory__IncorrectDataLength(uint256 expected, uint256 provided);

/// @dev The deployed implementation must be a contract.
/// @param implementation Implementation address.
- error ContractOnly(address implementation);
+ error StakingFactory__ContractOnly(address implementation);

/// @dev Proxy creation failed.
/// @param implementation Implementation address.
- error ProxyCreationFailed(address implementation);
+ error StakingFactory__ProxyCreationFailed(address implementation);

/// @dev Proxy instance initialization failed
/// @param instance Proxy instance address.
- error InitializationFailed(address instance);
+ error StakingFactory__InitializationFailed(address instance);

/// @dev Implementation is not verified.
/// @param implementation Implementation address.
- error UnverifiedImplementation(address implementation);
+ error StakingFactory__UnverifiedImplementation(address implementation);

/// @dev Proxy instance is not verified.
/// @param instance Proxy instance address.
- error UnverifiedProxy(address instance);
+ error StakingFactory__UnverifiedProxy(address instance);

/// @dev Caught reentrancy violation.
- error ReentrancyGuard();
+ error StakingFactory__ReentrancyGuard();

```
## [QA] Informational Findings on StakingNativeToken.sol
## [QA-5]  TITLE : QA Report for Error Functions
```diff
/// @dev Failure of a transfer.
/// @param token Address of a token.
/// @param from Address `from`.
/// @param to Address `to`.
/// @param value Value.
- error TransferFailed(address token, address from, address to, uint256 value);
+ error StakingNativeToken__TransferFailed(address token, address from, address to, uint256 value);
```

## [QA] Informational Findings on StakingActivityChecker.sol
## [QA-6]  TITLE : QA Report for Error Functions
```diff
// @dev Zero value when it has to be different from zero.
- error ZeroValue();
+ StakingActivityChecker__error ZeroValue();
```

## [QA] Informational Findings on SafeTransferLib.sol
## [QA-7]  TITLE : QA Report for Error Functions
```diff
/// @dev Failure of a token transfer.
/// @param token Address of a token.
/// @param from Address `from`.
/// @param to Address `to`.
/// @param value Value.
- error TokenTransferFailed(address token, address from, address to, uint256 value);
+  SafeTransferLib__error TokenTransferFailed(address token, address from, address to, uint256 value);
```

## [QA] Informational Findings on Dispenser.sol
## [QA-8]  TITLE : QA Report for Error Functions
```diff
/// @dev Only `manager` has a privilege, but the `sender` was provided.
/// @param sender Sender address.
/// @param manager Required sender address as a manager.
- error ManagerOnly(address sender, address manager);
+ error Dispenser__ManagerOnly(address sender, address manager);

/// @dev Only `owner` has a privilege, but the `sender` was provided.
/// @param sender Sender address.
/// @param owner Required sender address as an owner.
- error OwnerOnly(address sender, address owner);
+ error Dispenser__OwnerOnly(address sender, address owner);

/// @dev Provided zero address.
- error ZeroAddress();
+ error Dispenser__ZeroAddress();

/// @dev Wrong length of two arrays.
/// @param numValues1 Number of values in a first array.
/// @param numValues2 Number of values in a second array.
- error WrongArrayLength(uint256 numValues1, uint256 numValues2);
+ error Dispenser__WrongArrayLength(uint256 numValues1, uint256 numValues2);

/// @dev Zero value when it has to be different from zero.
- error ZeroValue();
+ error Dispenser__ZeroValue();

/// @dev Value overflow.
/// @param provided Overflow value.
/// @param max Maximum possible value.
- error Overflow(uint256 provided, uint256 max);
+ error Dispenser__Overflow(uint256 provided, uint256 max);

/// @dev Received lower value than the expected one.
/// @param provided Provided value is lower.
/// @param expected Expected value.
- error LowerThan(uint256 provided, uint256 expected);
+ error Dispenser__LowerThan(uint256 provided, uint256 expected);

/// @dev Wrong amount received / provided.
/// @param provided Provided amount.
/// @param expected Expected amount.
- error WrongAmount(uint256 provided, uint256 expected);
+ error Dispenser__WrongAmount(uint256 provided, uint256 expected);

/// @dev Caught reentrancy violation.
- error ReentrancyGuard();
+ error Dispenser__ReentrancyGuard();

/// @dev The contract is paused.
- error Paused();
+ error Dispenser__Paused();

/// @dev Incentives claim has failed.
/// @param account Account address.
/// @param reward Reward amount.
/// @param topUp Top-up amount.
- error ClaimIncentivesFailed(address account, uint256 reward, uint256 topUp);
+ error Dispenser__ClaimIncentivesFailed(address account, uint256 reward, uint256 topUp);

/// @dev Only the deposit processor is able to call the function.
/// @param sender Actual sender address.
/// @param depositProcessor Required deposit processor.
- error DepositProcessorOnly(address sender, address depositProcessor);
+ error Dispenser__DepositProcessorOnly(address sender, address depositProcessor);

/// @dev Chain Id is incorrect.
/// @param chainId Chain Id.
- error WrongChainId(uint256 chainId);
+ error Dispenser__WrongChainId(uint256 chainId);

/// @dev Account address is incorrect.
/// @param account Account address.
- error WrongAccount(bytes32 account);
+ error Dispenser__WrongAccount(bytes32 account);

```
***Conclusion:***
The error functions in this file are well-structured and provide clear, specific messages that facilitate debugging. By maintaining consistent usage and thorough documentation, the codebase can be further improved for better readability, maintainability, and functionality.





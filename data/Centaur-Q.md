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



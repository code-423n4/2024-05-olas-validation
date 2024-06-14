### Issue1
The structure field `maxStakingAmount` in `Dispenser`, and `maxStakingIncentive` in `Tokenomics` do not match together, leading to confusion.
```solidity
struct StakingPoint {
    // Amount of OLAS that funds service staking incentives for the epoch based on the inflation schedule
    // After 10 years, the OLAS inflation rate is 2% per year. It would take 220+ years to reach 2^96 - 1
    uint96 stakingIncentive;
    // Max allowed service staking incentive threshold
    // This value is never bigger than the stakingIncentive
    uint96 maxStakingIncentive;
    // Service staking vote weighting threshold
    // This number is bound by 10_000, ranging from 0 to 100% with the step of 0.01%
    uint16 minStakingWeight;
    // Service staking fraction
    // This number cannot be practically bigger than 100 as it sums up to 100% with others
    // maxBondFraction + topUpComponentFraction + topUpAgentFraction + stakingFraction <= 100%
    uint8 stakingFraction;
}
```
https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol#L239

```solidity
    struct StakingPoint {
        // Amount of OLAS that funds service staking for the epoch based on the inflation schedule
        // After 10 years, the OLAS inflation rate is 2% per year. It would take 220+ years to reach 2^96 - 1
        uint96 stakingIncentive;
        // Max allowed service staking incentive threshold
        // This value is never bigger than the stakingIncentive
        uint96 maxStakingAmount;
        // Service staking vote weighting threshold
        // This number is bound by 10_000, ranging from 0 to 100% with the step of 0.01%
        uint16 minStakingWeight;
        // Service staking fraction
        // This number cannot be practically bigger than 100 as it sums up to 100% with others
        // maxBondFraction + topUpComponentFraction + topUpAgentFraction + stakingFraction <= 100%
        uint8 stakingFraction;
    }
```
https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Dispenser.sol#L96

### Issue2
The revert message is not correct, and it should be `revert ManagerOnly(msg.sender, dispenser );` instead of `revert ManagerOnly(msg.sender, depository);`.
```solidity
        if (dispenser != msg.sender) {
            revert ManagerOnly(msg.sender, depository);
        }
```
https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/Tokenomics.sol#L829
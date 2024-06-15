# [L-01] Withdrawal revert for blacklisted user
Due to USDC blacklisting feature ,A user may not be able to claim their rewards if their account gets blacklisted
https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/registries/contracts/staking/StakingBase.sol#L508
https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/registries/contracts/staking/StakingToken.sol#L104
## Recommendation
Check for a blacklisting checking feature before claiming
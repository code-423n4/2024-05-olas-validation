# Utilize assembly to check that msg.sender is the owner 

We can utilize assembly to efficiently validate that msg.sender is equal to the owner using the smallest amount of opcodes possible. We can save 3 gas by using xor() instead of iszero(eq()). We could also save gas on reverts by using scratch space to store the error selector. This will potentially help avoid memory expansion costs. 

https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/governance/contracts/VoteWeighting.sol#L368

https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/Dispenser.sol#L636

https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/registries/contracts/staking/StakingFactory.sol#L110

https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/Tokenomics.sol#L546
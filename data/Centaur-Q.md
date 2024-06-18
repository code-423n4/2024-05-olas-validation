From the `TokenomicsConstants.sol` file, the `getSupplyCapForYear` function calculates the supply cap for a specific year based on the number of years passed from the launch date. There are a few errors in the code comment 
https://github.com/code-423n4/2024-05-olas/blob/3ce502ec8b475885b90668e617f3983cea3ae29f/tokenomics/contracts/TokenomicsConstants.sol#L28
1. Documentation Comment Correction:
The line 1e27 * (1.02)^(x-9) for x >= 10 should be corrected to 10e27 * (1.02)^(x-9) for x >= 10 to accurately reflect the logic used in the function. The initial supply cap for the 9th year is based on 1_000_000_000e18 which is 10e27.

2. Missing Documentation on Initial Supply Cap:
There is no explicit comment stating that the value at index zero in the supplyCaps array represents the initial supply cap at deployment. This should be added for clarity.

3. Comment Improvement:
The comment // For the first 10 years the supply caps are pre-defined should be enhanced to // For the first 10 years the supply caps are pre-defined, year of deployment inclusive to clearly indicate that the first year includes the deployment year


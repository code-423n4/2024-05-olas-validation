# Title
Cross-bridge leftovers and incorrectly sent funds will not be migrated

# Location
https://github.com/code-423n4/2024-05-olas/blob/main/tokenomics/contracts/staking/DefaultTargetDispenserL2.sol#L374-L455

# Vulnerability details
The function `migrate` in contract `DefaultTargetDispenserL2` is used to migrates funds to a new specified L2 target dispenser contract address. It will transfer all olas tokens to the new contract:
```solidity
// Get OLAS token amount
        uint256 amount = IToken(olas).balanceOf(address(this));
        // Transfer amount to the new L2 target dispenser
        if (amount > 0) {
            bool success = IToken(olas).transfer(newL2TargetDispenser, amount);
            if (!success) {
                revert TransferFailed(olas, address(this), newL2TargetDispenser, amount);
            }
        }
```
and then reset the `owner` to `address(0)`:
```solidity
// Zero the owner
        owner = address(0);
```
The issue is that the cross-bridge leftovers and incorrectly sent funds can not be migrated any more because the `drain()` function will check the owner address:
```solidity
if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }
```
Since the `owner` is set to `address(0)`, the `drain()` function will always revert and the cross-bridge leftovers and incorrectly sent funds in this contract will be locked forever.

# Recommendation
Consider sending  leftovers funds to the owner in function `migrate`:
```solidity
// Get OLAS token amount
        uint256 amount = IToken(olas).balanceOf(address(this));
        // Transfer amount to the new L2 target dispenser
        if (amount > 0) {
            bool success = IToken(olas).transfer(newL2TargetDispenser, amount);
            if (!success) {
                revert TransferFailed(olas, address(this), newL2TargetDispenser, amount);
            }
        }

        // Drain the slashed funds
        amount = address(this).balance;
        if (amount > 0) {
            // Send funds to the owner
        (bool success, ) = msg.sender.call{value: amount}("");
        if (!success) {
            revert TransferFailed(address(0), address(this), msg.sender, amount);
        }
        }

       
        // Zero the owner
        owner = address(0);
```
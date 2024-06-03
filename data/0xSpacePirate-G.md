### Receive function can be optimized

**Description:** 

The `StakingNativeToken::receive` uses too much gas and can be optimized.

**Impact:**
No impact, gas optimization

**Recommended Mitigation:**

The logic at the moment is the following:

```
    receive() external payable {
        // Add to the contract and available rewards balances
        uint256 newBalance = balance + msg.value;
        uint256 newAvailableRewards = availableRewards + msg.value;

        // Record the new actual balance and available rewards
        balance = newBalance;
        availableRewards = newAvailableRewards;

        emit Deposit(msg.sender, msg.value, newBalance, newAvailableRewards);
    }

```

Can be improved via assembly in the following way:

``` 
 receive() external payable {

        balance += msg.value;
        availableRewards += msg.value;

        //emit Deposit(msg.sender, msg.value, newBalance, availableRewards);

        assembly {
            mstore(0x00, availableRewards.slot)
            log4(
                0x00,
                0x20,
                // keccak256("Deposit(address,uint256,uint256,uint256)")
                0x36af321ec8d3c75236829c5317affd40ddb308863a1236d2d277a4025cccee1e,
                caller(),
                callvalue(),
                balance.slot
            )
        }
    }
```

Please note that apart from the assembly for the event, the newBalance and newAvailableRewards variables were redundant as well. 

In addition, the same approach with assembly can be repeated through the project for when it comes to emitting events which will save up even more gas overall.

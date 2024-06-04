### [GAS-0] Assembly for events and unnecessary other variables

**Description:** 

The `Receive` function in `StakingNativeToken::receive` uses too much gas and can be optimized.

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

Please note that apart from the assembly for the event, there was no need for the newBalance and newAvailableRewards variables as they were redundant.


**--------------------------------**

### [GAS-1] Unnecessary usage of memory instead of calldata

**Description:** 
In `GnosisTargetDispenserL2::receiveMessage` the usage of `memory` is redundant, instead using `calldata` would reduce the gas usage. This goes for all the other nested functions inside `GnosisTargetDispenserL2::receiveMessage`.

Since the `data` variables is passed through an external function `GnosisTargetDispenserL2::receiveMessage` we can use calldata and use it as well for all the other function being called inside the `GnosisTargetDispenserL2::receiveMessage` having in mind that the nested functions are not modifying the `data`.

**Impact:**
No impact, gas optimization

**Recommended Mitigation:**

In `GnosisTargetDispenserL2::receiveMessage` on line 87 of `GnosisTargetDispenserL2` would be better in the following way:

```

- function receiveMessage(bytes memory data) external {
+  function receiveMessage(bytes calldata data) external {
    ...
}

```

Inside the `GnosisTargetDispenserL2::receiveMessage` another function `_receiveMessage` is called using the same `data` variable. In `DefaultTargetDispenserL2::_receiveMessage` from line 224 to 228 in `DefaultTargetDispenserL2` would be better in the following way:

```

    function _receiveMessage(
        address messageRelayer,
        address sourceProcessor,
-        bytes memory data
+        bytes calldata data
    ) internal virtual { ... }

```

In addition, inside the already mentioned previous function `DefaultTargetDispenserL2::_receiveMessage` using the same variable `data` another function is called `_processData`. In `DefaultTargetDispenserL2::_processData` on line 139 of `DefaultTargetDispenserL2` would be better in the following way:

```
-    function _processData(bytes memory data) internal {
+    function _processData(bytes calldata data) internal { 
        ...
    }
```

**--------------------------------**


### [GAS-2] Usage of memory instead of calldata for and object related to initialization

In registries/contracts/staking `StakingToken::initialize` the usage of `memory` could be changed to `calldata` and reduce the gas usage. This goes as well for the nested `StakingBase::_initialize`.


**Description:** 

Since the `_stakingParams` variables are passed through an external function `StakingToken::initialize` we can use calldata.


**Impact:**
No impact, gas optimization

**Recommended Mitigation:**

In `StakingToken::initialize` on line 55 of `StakingToken` would be better in the following way:

```
    function initialize(
-        StakingParams memory _stakingParams,
+        StakingParams calldata _stakingParams,
        address _serviceRegistryTokenUtility,
        address _stakingToken
    ) external
    { ... }

```

Additionally, inside `StakingToken::initialize` we call `_initialize` which is a function in `StakingBase` contract in the same folder as `StakingToken`.

In `StakingBase::_initialize` on line 279 of `StakingBase` would be better in the following way:

```
    function _initialize(
-        StakingParams memory _stakingParams
+        StakingParams calldata _stakingParams
    ) internal
    { ... }

```

**--------------------------------**

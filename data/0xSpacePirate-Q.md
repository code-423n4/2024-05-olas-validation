
### Unnecessary usage of memory instead of calldata

**Description:** 
In `GnosisTargetDispenserL2::receiveMessage` the usage of `memory` is redundant, instead using `calldata` would reduce the gas usage. This goes for all the other nested functions inside `GnosisTargetDispenserL2::receiveMessage`.

Since the `data` variables is passed through an external function `GnosisTargetDispenserL2::receiveMessage` we can use calldata and use it as well for all the other function being called inside the `GnosisTargetDispenserL2::receiveMessage` having in mind that the nested functions are not modifying the `data`.

**Impact:**


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
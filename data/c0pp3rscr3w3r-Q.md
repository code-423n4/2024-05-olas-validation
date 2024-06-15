# QA 1
ChangeOwner can be made two step to avoid errors in setting Owner.

```solidity
 function changeOwner(address newOwner) external {
        // Check for the contract ownership
        if (msg.sender != owner) {
            revert OwnerOnly(msg.sender, owner);
        }

        // Check for the zero address
        if (newOwner == address(0)) {
            revert ZeroAddress();
        }

        owner = newOwner;
        emit OwnerUpdated(newOwner);
    }
```

## Recommended Mitigation steps
Use 2 step ownership instead of the one step of setting it to an arbitrary address.

________________________________________________________________________________________________________________________________
# QA 2
Wrong value emitted in Tokenomics

```solidity
    /// @param amount OLAS amount returned from staking.
    function refundFromStaking(uint256 amount) external {
        // Check for the dispenser access
        if (dispenser != msg.sender) {
            revert ManagerOnly(msg.sender, depository); //@audit wrong event parameter emitted
        }
```

## Recommended Mitigation steps

```diff
diff --git a/tokenomics/contracts/Tokenomics.sol b/tokenomics/contracts/Tokenomics.sol
index 6416cdd..e6610c6 100644
--- a/tokenomics/contracts/Tokenomics.sol
+++ b/tokenomics/contracts/Tokenomics.sol
@@ -826,7 +826,7 @@ contract Tokenomics is TokenomicsConstants {
     function refundFromStaking(uint256 amount) external {
         // Check for the dispenser access
         if (dispenser != msg.sender) {
-            revert ManagerOnly(msg.sender, depository);
+            revert ManagerOnly(msg.sender, dispenser); 
         }

         uint256 eCounter = epochCounter;
```

________________________________________________________________________________________________________________________________
# QA 3
Renundant checks can be made into modifiers
In `Dispenser.sol`

```solidity
        Pause currentPause = paused;
        if (currentPause == Pause.StakingIncentivesPaused || currentPause == Pause.AllPaused ||
            ITreasury(treasury).paused() == 2) {
            revert Paused();
        }
```

## Recommended Mitigation steps

can be made into

```solidity
modifier checkPauses(Pause pauseToCheck)
{
        Pause currentPause = paused;
        if (currentPause == pauseToCheck) || currentPause == Pause.AllPaused ||
            ITreasury(treasury).paused() == 2) {
            revert Paused();
        }
	_;

}
```


________________________________________________________________________________________________________________________________
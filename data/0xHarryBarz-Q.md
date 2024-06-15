Tokenomics.changeOwner allows the owner assign a new owner. Hence it would be more safe and considerate to consider adding a timelock to this function, should incase the owner wallet address is compromised by an attacker. A malicious function can be added to steal from users.


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

Adding a timelock would be safer.
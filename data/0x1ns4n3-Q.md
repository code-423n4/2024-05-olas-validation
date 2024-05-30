## ```++i``` costs less than ```i++```

++i costs less gas compared to i++ for unsigned integer, as pre-increment is cheaper (about 5 gas per iteration) i++ increments i and returns the initial value of i .

```
        for (uint256 i = 0; i < items.length; i++) {
            if (items[i] == item) {
                index = i;
                break;
            }
        }
```

So, ++i should be used instead of i++

link to code: https://github.com/code-423n4/2024-05-predy/blob/main/src/markets/gamma/ArrayLib.sol




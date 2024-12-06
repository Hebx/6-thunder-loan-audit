[M] The price of a token is determined by how many reserves are on either side of the pool.

**Impact**: Lp will drastically reduce fees for providing liquidity.

**PoC** : User takes a flash loan from `ThunderLoan` for 1000 `tokenA`. They are charged the original fee `fee1`. During the flash loan, they do the following:

1. User sells 1000 `tokenA`, tanking the price.
2. Instead of repaying right away, the user takes out another flash loan for another 1000 `tokenA`.
   1. Due to the fact that the way `ThunderLoan` calculates price based on the `TSwapPool` this second flash loan is substantially cheaper.

```
    function getPriceInWeth(address token) public view returns (uint256) {
        address swapPoolOfToken = IPoolFactory(s_poolFactory).getPool(token);
@>      return ITSwapPool(swapPoolOfToken).getPriceOfOnePoolTokenInWeth();
    }
```

The user then repays the first flash loan, and then repays the second flash loan.


**[H] By Calling a flashloan and then ThunderLoan::deposit instead of ThunderLoan::repay users can steal all funds from the protocol.**

__Description__:
user call a flashloan, deposited instead of repaying it (with the expected fee) will clear the flash loan and allow the user to redeem the amount of the loan ( stealing the funds) 
__Impact__
This exploit drains the liquidity pool from the flashloaned token

__Mitigation__:
Thunderloan could prevent deposits while an AssetToken is currently Flashloaning.
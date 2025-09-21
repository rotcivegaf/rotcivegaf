# Always passing `0` as the value of the `_chainlinkMaxRateTimespan` parameter of `swapTransferWithReference` and use deprecated chainlink functions

- Protocol: [Request Network](https://request.network/)
- Severity: Medium

## Target(s)

- [ChainlinkConversionPath](https://etherscan.io/address/0xC5519f3fcECC8EC85caaF8836563dEe9a00080f9#code)

## Impact(s)

- Direct theft of any user funds, whether at-rest or in-motion, other than unclaimed yield
- Miner-extractable value (MEV)
- Wrong request payment amount calculate

## Bug Description and impact

The `ERC20SwapToConversion` contract:

- [0x3b4837C9F4A606b71e61FD56Db6241781194df92](https://polygonscan.com/address/0x3b4837C9F4A606b71e61FD56Db6241781194df92)
- [0x3b4837C9F4A606b71e61FD56Db6241781194df92](https://etherscan.io/address/0x3b4837C9F4A606b71e61FD56Db6241781194df92)
- [0x80D1EE67ffAf7047d3E6EbF7317cF0eAd63FFc78](https://optimistic.etherscan.io/address/0x80D1EE67ffAf7047d3E6EbF7317cF0eAd63FFc78)
- [0x80D1EE67ffAf7047d3E6EbF7317cF0eAd63FFc78](https://moonscan.io/address/0x80D1EE67ffAf7047d3E6EbF7317cF0eAd63FFc78)

Has the `swapTransferWithReference` function that receives the `_chainlinkMaxRateTimespan` parameter, which is used to get the chainlink rates and then make payments

[The problem is that this function is always passing 0 as the value of the _chainlinkMaxRateTimespan parameter:](https://github.com/RequestNetwork/requestNetwork/blob/d539717009de82fe1195a73cb9f4ddbb0f59cd46/packages/payment-processor/src/payment/swap-any-to-erc20.ts#L170)

```typescript
  return swapToPayContract.interface.encodeFunctionData('swapTransferWithReference', [
    conversionProxyAddress,
    paymentAddress, // _to: string,
    amountToPay, // _requestAmount: BigNumberish,
    swapSettings.maxInputAmount, // _amountInMax: BigNumberish,
    swapSettings.path, // _uniswapPath: string[],
    path, // _chainlinkPath: string[],
    `0x${paymentReference}`, // _paymentReference: BytesLike,
    feeToPay, // _requestFeeAmount: BigNumberish,
    feeAddress || constants.AddressZero, // _feeAddress: string,
    Math.round(swapSettings.deadline / 1000), // _uniswapDeadline: BigNumberish,
    0, // _chainlinkMaxRateTimespan: BigNumberish,
  ]);
```

Causing the outdated rate protection of the `getConversions` function of the **Erc20ConversionProxy** contract to always pass:

```solidity
    require(
      _maxRateTimespan == 0 || block.timestamp - oldestTimestampRate <= _maxRateTimespan,
      'aggregator rate is outdated'
    );
```

This would bring errors to the calculation of amounts to be paid by users, since the price could be outdated

Another point of failure is that `_getConversion` also does not check the `oldestRateTimestamp` returned by `getConversion`:

```solidity
  function _getConversion(
    address[] memory _path,
    uint256 _requestAmount,
    uint256 _requestFeeAmount
  ) internal view returns (uint256 conversion) {
    (conversion, ) = chainlinkConversionPath.getConversion(
      _requestAmount + _requestFeeAmount,
      _path
    );
  }
```

Both points, makes the swap susceptible to MEV (sandwich attack) attacks, since the amount `paymentNetworkTotalAmount + requestSwapFeesAmount` can be wrong

Also, the contract `ChainlinkConversionPath` in the function `getRate` uses the deprecated functions [`latestTimestamp`](https://docs.chain.link/data-feeds/api-reference#latesttimestamp) and [`latestAnswer`](https://docs.chain.link/data-feeds/api-reference#latestanswer) from chainlink

```solidity
  /**
   * @notice Reads the current answer from aggregator delegated to.
   *
   * @dev #[deprecated] Use latestRoundData instead. This does not error if no
   * answer has been reached, it will simply return 0. Either wait to point to
   * an already answered Aggregator or use the recommended latestRoundData
   * instead which includes better verification information.
   */
  function latestAnswer()
    public
    view
    virtual
    override
    returns (int256 answer)
  {
    return currentPhase.aggregator.latestAnswer();
  }

  /**
   * @notice Reads the last updated height from aggregator delegated to.
   *
   * @dev #[deprecated] Use latestRoundData instead. This does not error if no
   * answer has been reached, it will simply return 0. Either wait to point to
   * an already answered Aggregator or use the recommended latestRoundData
   * instead which includes better verification information.
   */
  function latestTimestamp()
    public
    view
    virtual
    override
    returns (uint256 updatedAt)
  {
    return currentPhase.aggregator.latestTimestamp();
  }
```

## Recommendation

Use the [`latestRoundData`](https://docs.chain.link/data-feeds/api-reference#latestrounddata-1) of chainlink and check the return of this Instead of receiving the `_chainlinkMaxRateTimespan` by parameter, you could check that the delta of `updatedAt`(returned by `latestRoundData`) and `block.timestamp` is greater than a time delta, an hour as example:

```solidity
(, , , uint256 updatedAt, ) = aggregator.latestRoundData();
require(block.timestamp - updatedAt <= 1 hours, "aggregator rate is outdated");
```

## References

- [Chainlink api-reference](https://docs.chain.link/data-feeds/api-reference)
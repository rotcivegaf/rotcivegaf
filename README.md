## Solidity auditor and developer

### PoC of exploits:

| Protocol   | Stolen(USD) | Stolen     | The attacker use              | PoC |
|------------|-------------|------------|-------------------------------|-----|
|Penpiexyz_io|~$27.35M USD |11,113.6 ETH|Reentrancy-Reward Manipulation|[Penpiexyzio_exp.sol](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/8423a14b97998f1557d1216d340f605d31a6e99d/src/test/2024-09/Penpiexyzio_exp.sol)|
|OnyxDAO     |>$3.8M USD   |4.1M VUSD, 7.35M XCN, 5K DAI, 0.23 WBTC, 50K USDT|Flash loan-price manipulation-fake market|[OnyxDAO_exp](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/ef43599baa4d0b9dcd77eac49e4bda863d07d708/src/test/2024-09/OnyxDAO_exp.sol#L7-L20)|
|Bedrock_DeFi|~$1.7M USD   |27.84 BTC  |Swap ETH/BTC 1/1 in mint function|[Bedrock_DeFi_exp](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/4fb2da54f740df0def1927f1f0b7acf3087c02c3/src/test/2024-09/Bedrock_DeFi_exp.sol#L37-L47)|
|P719Token  |~$312K USD|547.18 BNB|Flash loans-price manipulation|[P719Token_exp](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/82a4a71c7a37a67d13a97e072a8cf42c167603c3/src/test/2024-10/P719Token_exp.sol#L7-L21)|
|LavaLending |~$130K USD   |1 USDC, 125795.6 cUSDC, 0,0067 WBTC, 2.25 WETH|5 Flash loans-price manipulation|[LavaLending_exp](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/b3cda5b5c08d453ef96dcb2e7ea44d7a40e7a85e/src/test/2024-10/LavaLending_exp.sol)|
|FIREToken   |~$20K USD|8.45 ETH|A flash loan-pair manipulation with the `_transfer`|[FireToken_exp](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/2fdff2671591c251ba9a514afbda6bc0aac03e32/src/test/2024-10/FireToken_exp.sol#L7-L16)|
|AIZPTToken  |~$20K USD|34.88 BNB|Flash loans-wrong price calculation|[AIZPTToken_exp](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/014e23d0ebc9c8563e772d27672f05ed2063b36f/src/test/2024-10/AIZPTToken_exp.sol#L7C28-L12)|
 
## Audits reports

### Profiles:
- [Hats Finance](https://app.hats.finance/profile/rotcivegaf)
- TODO

### Reports

| Date       | Company  | Protocol   | Severity | Report |
|------------|----------|------------|:--------:|:------:|
| 2022/12/06 | Immunefi | Thena      | Low      | [:page_facing_up:](reports/2022-12-06-Thena.md) |


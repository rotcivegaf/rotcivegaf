## Solidity auditor and developer

### PoC of exploits:

- **@LavaLending**
  - Total stolen: 1 USDC, 125795.6 cUSDC, 0,0067 WBTC, 2.25 WETH (~$130K USD)
  - The attacker use: 5 Flash loans and price manipulation
  - [PoC Exploit](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/b3cda5b5c08d453ef96dcb2e7ea44d7a40e7a85e/src/test/2024-10/LavaLending_exp.sol)

- **@OnyxDAO**
  - Total stolen: 4.1M VUSD, 7.35M XCN, 5K DAI, 0.23 WBTC, 50K USDT (>$3.8M USD)
  - The attacker use: Flash loan, price manipulation and fake market; 5 contracts in total
  - [PoC Exploit](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/ef43599baa4d0b9dcd77eac49e4bda863d07d708/src/test/2024-09/OnyxDAO_exp.sol#L7-L20)
  
- **@Bedrock_DeFi**
  - Total stolen: 27.83925883 BTC (~$1.7M USD)
  - The attacker use: Swap ETH/BTC 1/1 in mint function
  - [PoC Exploit](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/4fb2da54f740df0def1927f1f0b7acf3087c02c3/src/test/2024-09/Bedrock_DeFi_exp.sol#L37-L47)

- **@Penpiexyz_io**
  - Total stolen: 11,113.6 ETH (~$27,348,259 USD)
  - The attacker use: Reentrancy and Reward Manipulation
  - [PoC Exploit](https://github.com/SunWeb3Sec/DeFiHackLabs/blob/8423a14b97998f1557d1216d340f605d31a6e99d/src/test/2024-09/Penpiexyzio_exp.sol)


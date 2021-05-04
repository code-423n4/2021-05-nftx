# ✨ So you want to sponsor a contest

This `README.md` contains a set of checklists for our contest collaboration.

Your contest will use two repos: 
- **a _contest_ repo** (this one), which is used for scoping your contest and for providing information to contestants (wardens)
- **a _findings_ repo**, where issues are submitted. (We'll set that one up later.) 

Ultimately, when we launch the contest, this contest repo will be made public and will contain the smart contracts to be reviewed and all the information needed for contest participants. The findings repo will be made public after the contest is over and your team has mitigated the identified issues.

Some of the checklists in this doc are for **C4 (🐺)** and some of them are for **you as the contest sponsor (⭐️)**.

---

# Contest prep

## 🐺 C4: Contest prep
- [ ] Rename contest H1 below
- [ ] Add link to report form in contest details below
- [ ] Update pot sizes
- [ ] Fill in start and end times in contest bullets below.
- [ ] Move any relevant information in "contest scope information" above to the bottom of this readme.
- [ ] Add matching info to the [code423n4.com public contest data here](https://github.com/code-423n4/code423n4.com/tree/main/data/contests))
- [ ] Delete this checklist.

## ⭐️ Sponsor: Contest prep
- [ ] Modify the bottom of this `README.md` file to describe how your code is supposed to work with links to any relevent documentation and any other criteria/details that the C4 Wardens should keep in mind when reviewing
- [ ] Describe any novel or unique curve logic or mathematical models implemented in the contracts
- [ ] Does the token conform to the ERC-20 standard? In what specific ways does it differ?
- [ ] Describe anything else that adds any special logic that makes your approach unique
- [ ] Identify any areas of specific concern in reviewing the code
- [ ] Add all of the code to this repo that you want reviewed
- [ ] Make sure your code is thoroughly commented using the [NatSpec format](https://docs.soliditylang.org/en/v0.5.10/natspec-format.html#natspec-format).
- [ ] Please have final versions of contracts and documentation added/updated in this repo **no less than 8 hours prior to contest start time.**
- [ ] Ensure that you have access to the _findings_ repo where issues will be submitted.
- [ ] Delete this checklist and all text above the line below when you're ready.

---

# NFTX contest details
- ~$60K (ETH) main award pot
- ~$6K (ETH) gas optimization award pot
- Join [C4 Discord](https://discord.gg/EY5dvm3evD) to register
- Submit findings [using the C4 form](https://c4-nftx.netlify.app/)
- [Read our guidelines for more details](https://code423n4.com/compete)
- Starts May 5 00:00 UTC
- Ends May 11 23:59 UTC

This repo will be made public before the start of the contest.

## Overview
NFTX is a protocol for making tokenized baskets of similarly priced NFTs. Occasionally these baskets have been called funds but are officially referred to as vaults. Right now with version 1, all NFTX vaults have their state stored together but have seperate contracts for every ERC20 vault token (vtoken), however version 2 is designed so that every ERC20 vtoken also stores its own vault state and implements necessary vault functions. In this sense NFTXv2 combines vault logic and vault token logic into a single contract which is simply refferred to as a vault. These vaults get deployed using a vault factory contract.

The primary vault operations are minting and redeeming. Minting takes an NFT from the user and gives a vToken in return. For example, in order for a user to mint 1 GLYPH token it is necessary that they give ownership of 1 autoglyph NFT to the vault. Redeeming is the opposite and requires that the user gives 1 vToken in order to receive 1 NFT (e.g. 1 GLYPH token for 1 autoglyph). Redeeming is random by default, but it is possible for specific NFT tokenIds to be inputed as a parameter to the redeem function. We refer to specific redeems as direct redeems. Both minting and redeeming can happen individually or in bulk, and both ERC721 and ERC1155 are supported. 

The account which calls createVault on the factory contract gets designated as the vault manager during the vault's deployment and can then customize the vault's settings to their liking after it has been initialized. It is possible for the manager to toggle vault operations, set fees, and set custom eligibility preferences. When the manager is done customizing the vault they can then call a finalize function which renounces their control. 

When vaults are deployed they are initially set to either allow all tokenIDs or to allow zero tokenIDs. Vaults which allow all tokenIDs are known as floor vaults. Vaults which allow no tokenIDs act as a blank canvas for vault managers to deploy what is called an eligibility module. There are different eligibility modules for different usecases. Each vault can have at most one eligibility module, but it is possible for custom eligibility modules to be developped and deployed manually.

## Assumptions
You may assume that all NFT contracts are built in good faith and comply with either the ERC721 or ERC1155 spec.

## Areas of Review
Vaults should *always* maintain 1:1 ratio between vToken supply and vault holdings. For example, if the supply of GLYPH is 42 then there should be exactly 42 autoglyphs owned by the vault contract. This rule also applies to ERC1155 collections, however since it's possible for each 1155 tokenID to have multiple copies, it is the sum of all tokenID balances which must equal the supply of the vToken (e.g. if there is an ERC1155 collection called CryptoPandas and there is an NFTX vault with the symbol PANDA and a supply of 7, then it would be possible for the vault to hold tokenIDs 123 and 132 with balances of 3 and 4, because 3 + 4 = 7).

Vault settings should only be configurable by the vault manager or the contract owner. When the vault manager is set to a non-zero address then it should be the only account which can modify settings. When the vault manager is set to the zero address, then control should be deferred to the contract owner (which will be the NFTX Dao). 

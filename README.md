# ✨ So you want to sponsor a contest

This `README.md` contains a set of checklists for our contest collaboration.

Your contest will use two repos: 
- **a _contest_ repo** (this one), which is used for scoping your contest and for providing information to contestants (wardens)
- **a _findings_ repo**, where issues are submitted (shared with you after the contest) 

Ultimately, when we launch the contest, this contest repo will be made public and will contain the smart contracts to be reviewed and all the information needed for contest participants. The findings repo will be made public after the contest report is published and your team has mitigated the identified issues.

Some of the checklists in this doc are for **C4 (🐺)** and some of them are for **you as the contest sponsor (⭐️)**.

---

# Repo setup

## ⭐️ Sponsor: Add code to this repo

- [x] Create a PR to this repo with the below changes:
- [x] Provide a self-contained repository with working commands that will build (at least) all in-scope contracts, and commands that will run tests producing gas reports for the relevant contracts.
- [x] Make sure your code is thoroughly commented using the [NatSpec format](https://docs.soliditylang.org/en/v0.5.10/natspec-format.html#natspec-format).
- [x] Please have final versions of contracts and documentation added/updated in this repo **no less than 24 hours prior to contest start time.**
- [x] Be prepared for a 🚨code freeze🚨 for the duration of the contest — important because it establishes a level playing field. We want to ensure everyone's looking at the same code, no matter when they look during the contest. (Note: this includes your own repo, since a PR can leak alpha to our wardens!)


---

## ⭐️ Sponsor: Edit this README

Under "SPONSORS ADD INFO HERE" heading below, include the following:

- [x] Modify the bottom of this `README.md` file to describe how your code is supposed to work with links to any relevent documentation and any other criteria/details that the C4 Wardens should keep in mind when reviewing. ([Here's a well-constructed example.](https://github.com/code-423n4/2022-08-foundation#readme))
  - [x] When linking, please provide all links as full absolute links versus relative links
  - [x] All information should be provided in markdown format (HTML does not render on Code4rena.com)
- [x] Under the "Scope" heading, provide the name of each contract and:
  - [x] source lines of code (excluding blank lines and comments) in each
  - [x] external contracts called in each
  - [x] libraries used in each
- [x] Describe any novel or unique curve logic or mathematical models implemented in the contracts
- [ ] Does the token conform to the ERC-20 standard? In what specific ways does it differ?
- [ ] Describe anything else that adds any special logic that makes your approach unique
- [ ] Identify any areas of specific concern in reviewing the code
- [ ] Optional / nice to have: pre-record a high-level overview of your protocol (not just specific smart contract functions). This saves wardens a lot of time wading through documentation.
- [ ] Delete this checklist and all text above the line below when you're ready.

---

# Forgeries contest details
- Total Prize Pool: $36,500 USDC
  - HM awards: $25,500 USDC 
  - QA report awards: $3,000 USDC 
  - Gas report awards: $1,500 USDC 
  - Judge + presort awards: $6,000 USDC 
  - Scout awards: $500 USDC
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2022-12-forgeries-contest/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts December 09, 2022 20:00 UTC
- Ends December 12, 2022 20:00 UTC

## C4udit / Publicly Known Issues

The C4audit output for the contest can be found [here](add link to report) within an hour of contest opening.

*Note for C4 wardens: Anything included in the C4udit output is considered a publicly known issue and is ineligible for awards.*

We rely on `@chainlink/contracts` to supply `VRF` numbers and this contract clearly documents `chainlink` as a dependency for this project. Any issues directly related to the chainlink infastructure contracts other than incorrect configuration of their libraries within this project are not in scope for this audit.

# Overview

We want to raffle away a single NFT based off of another NFT collection (or _ticket_ contract).

For instance, we could raffle off a single high value NFT to any cryptopunk holder, the punk that wins can choose to claim the NFT. If they do not claim, a re-roll or redraw can be done to select a new holder that would be able to claim the NFT.

The contract is a hyperstructure (https://jacob.energy/hyperstructures.html) except for the dependency on chain.link (https://chain.link/).

We are utilizing the `chain.link` Verifiable Random Function (`VRF`) contract tools to fairly raffle off the NFT. Their `VRF` docs can be found at: https://docs.chain.link/vrf/v2/introduction/.

The main functions are `VRFNFTRandomDrawFactory.makeNewDraw()` to create a new non-upgradeable minimal clones proxy draw contract with your desired configuration. Each contract is seperate to allow for easier UX and more security with interactions. After the drawing is created, it needs to be started which will pull the NFT from the creator/owner's wallet up for raffle when they call `VRFNFTRandomDraw.startDraw()`.

After the drawing is started, we will request a random entropy from chain.link using the internal `_requestRoll()` function. Once chain.link returns the data in the `fulfillRandomWords()` callback the raffle NFT will be chosen and saved. If the raffle NFT is burned or removed this will still complete and a redraw will need to happen to find an NFT that is active/accessible to draw the winning NFT. Most raffles will use a specific contract that users will have a high incentive to withdraw their winning NFT.

The winning user can determine if they have won by calling `hasUserWon(address)` that checks the owner of the winning NFT to return the winning user. They also can look at `request().currentChosenTokenId` to see the currently chosen winning NFT token id. Once they have won, they can call `winnerClaimNFT()` from the account that won to have the raffled NFT transferred to the winner.

If the winning user does not claim the winning NFT within a specific deadline, the owner can call `redraw()` to redraw the NFT raffle. This is an `ownerOnly` function that will call into chainlink.

If no users ultimately claim the NFT, the admin specifies a timelock period after which they can retrieve the raffled NFT.

# Scope

*List all files in scope in the table below -- and feel free to add notes here to emphasize areas of focus.*

| Contract | SLOC | Purpose | Libraries used |  
| ----------- | ----------- | ----------- | ----------- |
| IVRFNFTRandomDraw | 10 | Interface to the factory VRFNFTRandomDrawFactory contract | None |
| IVRFNFTRandomDrawFactory | 62 | Interface to the main VRFNFTRandomDraw contract | None |
| VRFNFTRandomDrawFactory | 27 | This contract is a factory for raffles, allows creating a new drawing contract. | [`@openzeppelin/contracts-upgradeable`](https://openzeppelin.com/contracts/) |
| VRFNFTRandomDraw | 194 | This contract is the main escrow and VRF-integrated raffle contract | [`@openzeppelin/contracts-upgradeable`](https://openzeppelin.com/contracts/), [`@chainlink/contracts`](https://github.com/smartcontractkit/chainlink) |
| IOwnableUpgradeable | 15 | The interface to an owner safe-transferrable upgradeable openzeppelin fork | | 
| OwnableUpgradeable | 73 | This contract is the main escrow and VRF-integrated raffle contract | [`@openzeppelin/contracts-upgradeable`](https://openzeppelin.com/contracts/) |

## Out of scope

*List any files/contracts that are out of scope for this audit.*

1. Openzeppelin dependency contracts
2. Chainlink depedency architecture / contracts
3. Issues / drawbacks of using specific EIP standards (EIP721 (NFT Token standard), EIP1167 (minimal proxies/clones))
4. Assume NFT up for raffle and NFT that is used for the raffle are both non-malicious contract (While this is good to note)

# Additional Context

1. We have no unique curve logic or mathmetical models in this contract.

## Scoping Details 
```
- If you have a public code repo, please share it here:  https://github.com/0xigami/vrf-nft-raffle
- How many contracts are in scope?:   2
- Total SLoC for these contracts?:  250
- How many external imports are there?: 6 
- How many separate interfaces and struct definitions are there for the contracts within scope?:  2 structs, 0 interfaces
- Does most of your code generally use composition or inheritance?:   Code typically uses composition
- How many external calls?:   4
- What is the overall line coverage percentage provided by your tests?:  94
- Is there a need to understand a separate part of the codebase / get context in order to audit this part of the protocol?:   false
- Please describe required context:   
- Does it use an oracle?:  true
- Does the token conform to the ERC20 standard?:  N/A
- Are there any novel or unique curve logic or mathematical models?: N/A
- Does it use a timelock function?:  Yes
- Is it an NFT?: No (but NFTs get locked inside)
- Does it have an AMM?:   No
- Is it a fork of a popular project?:   false
- Does it use rollups?:   false
- Is it multi-chain?:  false
- Does it use a side-chain?: false 
```

# Tests

*Provide every step required to build the project from a fresh git clone, as well as steps to run the tests with a gas report.* 

To run tests:
1. Setup `yarn` and `forge`
1. Install dependencies: `yarn`
2. Run tests: `yarn test`

Slither notes:
1. Slither will have issues with the try/catch blocks upon first test
2. All test files and files from `chain.link` have issues that are out of scope in the repo for Slither.

*Note: Many wardens run Slither as a first pass for testing.  Please document any known errors with no workaround.* 

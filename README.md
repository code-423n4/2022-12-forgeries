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

We rely on `@chainlink/contracts` to supply `VRF` numbers and this contract clearly documents `chain.link` as a dependency for this project. Any issues directly related to the chainlink infastructure contracts other than incorrect configuration of their libraries within this project are not in scope for this audit.

# Overview

We want to raffle away a single NFT (_token_) based off of another NFT collection (or _drawingToken_) in a fair and trustless manner.

For instance, we could raffle off a single high value NFT to any cryptopunk holder, the punk that wins can choose to claim the NFT. If they do not claim, a re-roll or redraw can be done to select a new holder that would be able to claim the NFT.

The contract follows the hyperstructure concept (https://jacob.energy/hyperstructures.html) except for the dependency on chain.link (https://chain.link/).

We are utilizing the `chain.link` Verifiable Random Function (`VRF`) contract tools to fairly raffle off the NFT. Their `VRF` docs can be found at: https://docs.chain.link/vrf/v2/introduction/.

The main functions are `VRFNFTRandomDrawFactory.makeNewDraw()` to create a new non-upgradeable minimal clones proxy draw contract with your desired configuration. Each contract is separate to allow for easier UX and more security with interactions. After the drawing is created, it needs to be started which will pull the NFT from the creator/owner's wallet up for raffle when they call `VRFNFTRandomDraw.startDraw()`.

After the drawing is started, we will request a random entropy from chain.link using the internal `_requestRoll()` function. Once `chain.link` returns the data in the `fulfillRandomWords()` callback the raffle NFT will be chosen and saved. If the raffle NFT is burned or removed this will still complete and a redraw will need to happen to find an NFT that is active/accessible to draw the winning NFT. Most raffles will use a specific contract that users will have a high incentive to withdraw their winning NFT.

The winning user can determine if they have won by calling `hasUserWon(address)` that checks the owner of the winning NFT to return the winning user. They also can look at `request().currentChosenTokenId` to see the currently chosen winning NFT token id. Once they have won, they can call `winnerClaimNFT()` from the account that won to have the raffled NFT transferred to the winner.

If the winning user does not claim the winning NFT within a specific deadline, the owner can call `redraw()` to redraw the NFT raffle. This is an `ownerOnly` function that will call into `chain.link`.

If no users ultimately claim the NFT, the admin specifies a timelock period after which they can retrieve the raffled NFT.

# Scope

| Contract | SLOC | Purpose | Libraries used |  
| ----------- | ----------- | ----------- | ----------- |
| IVRFNFTRandomDraw | 10 | Interface to the factory VRFNFTRandomDrawFactory contract | None |
| IVRFNFTRandomDrawFactory | 62 | Interface to the main VRFNFTRandomDraw contract | None |
| IVRFNFTRandomDrawFactoryProxy | 62 | Interface to the main VRFNFTRandomDraw contract | None |
| VRFNFTRandomDrawFactory | 41 | Factory for VRF NFT Raffle, UUPS Upgradable by owner. | [`@openzeppelin/contracts-upgradeable`](https://openzeppelin.com/contracts/) |
| VRFNFTRandomDrawFactoryProxy | 41 | Proxy Contract linking to the Factory | [`@openzeppelin/contracts-upgradeable`](https://openzeppelin.com/contracts/) |
| VRFNFTRandomDraw | 194 | This contract is the main escrow and VRF-integrated raffle contract | [`@openzeppelin/contracts-upgradeable`](https://openzeppelin.com/contracts/), [`@chainlink/contracts`](https://github.com/smartcontractkit/chainlink) |
| IOwnableUpgradeable | 15 | The interface to an owner safe-transferrable upgradeable openzeppelin fork | | 
| OwnableUpgradeable | 73 | This contract is the main escrow and VRF-integrated raffle contract | [`@openzeppelin/contracts-upgradeable`](https://openzeppelin.com/contracts/) |

## Out of scope

1. OpenZeppelin dependency contracts
2. UUPS Proxy and OpenZeppelin implementation thereof
3. Chainlink dependency architecture / contracts
4. Issues / drawbacks of using specific EIP standards (EIP721 (NFT Token standard), EIP1167 (minimal proxies/clones))
5. The NFT up for raffle and NFT that is used for the raffle are non-malicious contracts not attempting to compromise this raffle contract (within reason). Assume creators of raffles will do checks to ensure that the NFT itself is not compromised or unusual preventing the functioning of the raffle contract.

# Additional Context

1. We have no unique curve logic or mathematical models in this contract.
2. This contract should prevent the owner from iterrupting the contest until the timelock unlocks at the end and preventing any users that do not win from withdrawing the NFT.

## Scoping Details 
```
- If you have a public code repo, please share it here:  https://github.com/0xigami/vrf-nft-raffle
- How many contracts are in scope?:   3
- Total SLoC for these contracts?:  320
- How many external imports are there?: 6 
- How many separate interfaces and struct definitions are there for the contracts within scope?:  2 structs, 2 interfaces
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

### To run tests:

1. Setup `yarn`: https://yarnpkg.com/getting-started/install
2. Setup `forge`: https://book.getfoundry.sh/getting-started/installation
3. Install dependencies: run `yarn`
4. Run tests: run `yarn test`

### Slither notes:

1. Slither will have issues with the try/catch blocks upon first test
2. All test files and files from `chain.link` have issues that are out of scope in the repo for Slither.
# Gitcoin's Allo Protocol Contest Details

- Join [Sherlock Discord](https://discord.gg/MABEWyASkp)
- Submit findings using the issue page in your private contest repo (label issues as med or high)
- [Read for more details](https://docs.sherlock.xyz/audits/watsons)

# Overview

Allo Protocol has three main components to it:

1. **A Project Registry.** This is the universal, on-chain registry of projects. To
   apply to be a grantee in any round, a project has to first register in the Project
   Registry.
2. **Programs.** Each grant-giving organization will have an on-chain program, which
   manages it's rounds.
3. **Rounds.** A pool of funds to be distributed based on a custom voting
   strategy and payout strategy.

## Project Registry

Before applying to participate in a grants round, a project (a grant-receiving
wallet address, representing an individual or a group) must register in the
Project Registry.

## Rounds

Funding to be distributed through Allo Protocol are held in the Round, which
consists of a voting strategy and a payout strategy.
A voting strategy is a smart
contract that encodes how a community would like to decide to allocate the
funds held in a round. 
A payout strategy is a smart contract that controls how
the funds are distributed when the round ends.

Note: 
- Funds are originally sent to the round.
- The funds are sent from the round to payout contract after the 
  distribuion is final and round has ended
- Fund can be native / ERC20 token

### Voting Strategy

Gitcoin uses and has provided an implementation of a Quadratic Funding Voting
Strategy (in-scope for the audit). But the idea is that anyone could write
a custom voting strategy that adheres to the `IVotingStrategy` (also provided).
Some common voting strategies that we expect to emerge include:

- A simple token voting strategy (1 token, 1 vote)
- Other forms of quadratic voting
- Approval voting (Any vote earns a payout)

### Payout Strategy

Gitcoin pays out it's grantees using the provided Merkle Payout Strategy. We
calculate the pool distribution off-chain and add it to the payout strategy
contract as a Merkle tree.

This is one implementation of how quadratic funding could be paid out. We
imagine that others will emerge, including claim-based instead of payout-based.
We also imagine other types payouts will be implemented as payout strategies,
including:

- Direct grants: user receives a fixed grant amount when the round ends
- Sweepstakes/contest: top-voted projects receive allocations of the pool

# Resources

- [Documentation](https://docs.allo.gitcoin.co/getting-started/introduction)
- [Background](https://gov.gitcoin.co/t/introducing-gitcoin-grants-stack-allo-protocol-product-overviews-part-1-of-2/12664#the-story-of-allo-protocol-rebuilding-cgrants-from-the-ground-up-1)

# On-chain context

**Some pointers for filling out the section below:**  
ERC20/ERC721/ERC777/FEE-ON-TRANSFER/REBASING TOKENS:  
*Which tokens do you expect will interact with the smart contracts? Please note that these answers have a significant impact on the issues that will be submitted by Watsons. Please list specific tokens (ETH, USDC, DAI) where possible, otherwise "Any"/"None" type answers are acceptable as well.*

ADMIN:
*Admin/owner of the protocol/contracts.
Label as TRUSTED, If you **don't** want to receive issues about the admin of the contract being able to steal funds. 
If you want to receive issues about the Admin of the contract being able to steal funds, label as RESTRICTED & list specific acceptable/unacceptable actions for the admins.*

EXTERNAL ADMIN:
*These are admins of the protocols your contracts integrate with (if any). 
If you **don't** want to receive issues about this Admin being able to steal funds or result in loss of funds, label as TRUSTED
If you want to receive issues about this admin being able to steal or result in loss of funds, label as RESTRICTED.*
 
```
DEPLOYMENT: Ethereum mainnet, Fantom, 
ERC20: any
ERC721: none
ERC777: none
FEE-ON-TRANSFER: none
REBASING TOKENS: none
ADMIN: contract owner
EXTERNAL-ADMINS: trusted
```


### Q: Are there any additional protocol roles? If yes, please explain in detail:
1) The roles
2) The actions those roles can take 
3) Outcomes that are expected from those roles 
4) Specific actions/outcomes NOT intended to be possible for those roles

A: Payout Strategy Round Operator

In a Payout Strategy, the Round Operator is a community-trusted role, set when
the Payout Strategy contract is deployed.

Certain actions in the Payout Strategy are restricted to addresses with the
Round Operator role. This includes 
- updating the payout distribution (`updateDistribution()`) 
- withdrawing any extra funds from the matching pool(`withdrawFunds()`)
- paying out the projects (`payout()`)

B: Program Implementation Program Operator

A Program represents a grant-giving organization on-chain, one that creates
Rounds. On the Program Implementation contract, updating the metadata for
a program is restricted to Program Operators.

C: Round Implementation Round Operator

In a Round contract, an address can be given certain administrative privileges
on the round by giving it the Round Operator role. This is how we restrict all
of the parameter setter methods. This role is also how we restrict a couple of
key functional methods on the round:

- `setReadyForPayout()`: This method, called when the round ends, pays the
    protocol fee and transfers the matching pool to the Payout Contract, where
    it is held until funds are distributed.
- `withdraw()`: Used to withdraw ("rescue") any funds transferred to the
    matching pool by mistake that are not meant to be distributed in the grant
    round

Other functions around upating the round are restricted to the Round Operator
role and cannot be invoked after the round ends which include
- updating the round metadata, start - end data
- updating the round amount  

___
### Q: Are there any off-chain mechanisms or off-chain procedures for the protocol (keeper bots, input validation expectations, etc)? 
A: Distributions in quadratic funding rounds are calculated off-chain, based on
votes in the Quadratic Funding Voting Strategy. This is for three reasons. First,
calculating the distribution would be really expensive. Second, grantees need to
be KYC-ed before they can receive grant funding. Third, we have to 'validate'
votes and remove any Sybil attacks before counting votes in the quadratic curve
distribution.

# Audit scope

[contracts @ 36dc33762c396660c0a84f6ef7d790f632638e81](https://github.com/allo-protocol/contracts/tree/36dc33762c396660c0a84f6ef7d790f632638e81)
- [contracts/contracts/payoutStrategy/IPayoutStrategy.sol](contracts/contracts/payoutStrategy/IPayoutStrategy.sol)
- [contracts/contracts/payoutStrategy/MerklePayoutStrategy/MerklePayoutStrategyFactory.sol](contracts/contracts/payoutStrategy/MerklePayoutStrategy/MerklePayoutStrategyFactory.sol)
- [contracts/contracts/payoutStrategy/MerklePayoutStrategy/MerklePayoutStrategyImplementation.sol](contracts/contracts/payoutStrategy/MerklePayoutStrategy/MerklePayoutStrategyImplementation.sol)
- [contracts/contracts/program/ProgramFactory.sol](contracts/contracts/program/ProgramFactory.sol)
- [contracts/contracts/program/ProgramImplementation.sol](contracts/contracts/program/ProgramImplementation.sol)
- [contracts/contracts/projectRegistry/ProjectRegistry.sol](contracts/contracts/projectRegistry/ProjectRegistry.sol)
- [contracts/contracts/round/IRoundFactory.sol](contracts/contracts/round/IRoundFactory.sol)
- [contracts/contracts/round/IRoundImplementation.sol](contracts/contracts/round/IRoundImplementation.sol)
- [contracts/contracts/round/RoundFactory.sol](contracts/contracts/round/RoundFactory.sol)
- [contracts/contracts/round/RoundImplementation.sol](contracts/contracts/round/RoundImplementation.sol)
- [contracts/contracts/settings/AlloSettings.sol](contracts/contracts/settings/AlloSettings.sol)
- [contracts/contracts/utils/MetaPtr.sol](contracts/contracts/utils/MetaPtr.sol)
- [contracts/contracts/votingStrategy/IVotingStrategy.sol](contracts/contracts/votingStrategy/IVotingStrategy.sol)
- [contracts/contracts/votingStrategy/QuadraticFundingStrategy/QuadraticFundingVotingStrategyFactory.sol](contracts/contracts/votingStrategy/QuadraticFundingStrategy/QuadraticFundingVotingStrategyFactory.sol)
- [contracts/contracts/votingStrategy/QuadraticFundingStrategy/QuadraticFundingVotingStrategyImplementation.sol](contracts/contracts/votingStrategy/QuadraticFundingStrategy/QuadraticFundingVotingStrategyImplementation.sol)

# About Gitcoin's Allo Protocol

Allo Protocol represents the next iteration of Gitcoin's public goods funding.
These smart contracts originated in a product called cGrants, Gitcoin's
centralized public goods product that has hosted multiple quadratic funding
rounds and distributed about $50 million of funding to the Ethereum ecosystem.

Now, Gitcoin is decentralizing. We will continue to run our grants program,
while we empower other communities to fund their shared needs through two key
initiatives:

1. The Grants Stack: a full-service product suite for running a grants program
   from end-to-end
2. Allo Protocol: a protocol for the on-chain allocation of capital

Allo Protocol, specifically, allows communities to build their own custom
mechanisms for (a) deciding how to distribute capital and (b) payout out
capital.

**Further Reading:**

- [Introducing: Gitcoin Grants Stack & Allo Protocol - Product Overviews (Part 1 of 2)](https://gov.gitcoin.co/t/introducing-gitcoin-grants-stack-allo-protocol-product-overviews-part-1-of-2/12664)
- [Announce the decentralized future of Gitcoin Grants](https://go.gitcoin.co/blog/announcing-the-decentralized-future-of-gitcoin-grants)
- [Liberal Radicalism: A Flexible Design For Philanthropic Matching Funds](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3243656)

# Team & Contact

We want to make this contest as smooth as possible for you. Please ask a lot of
questions and let us know if the documentation or system is unclear in any way.

We'll be in the Discord with DMs open:

- dev: @thelostone-mc#7333
- dev: @gravityblast#9625
- dev: @zakk | Gitcoin Dev Ecosystem#0896
- pm: @heynate#9251

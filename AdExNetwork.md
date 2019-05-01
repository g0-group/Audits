# Audit report
## AdEx Protocol
Repositories
- https://github.com/AdExNetwork/adex-protocol/commit/4e5794bc837f69ee1741ff8c1ec5112edecf5197

## Files

All solidity files included in the repository

## Issues

### AdEx Core

### Introduction and general assesment

To clarify security properties of AdEx OUTPACE protocol, we'll look at L2 solutions in general and contrast some specific ones with OUTPACE.

In general blockchains provide two services, the first one is cryptoeconomic assurance of transaction ordering and the second is assurance of data integrity. 

Transaction ordering is crucial for protocols that allow for mutually exclusive state transistions. The most basic example illustrating why is the double spending attack. If owner of a property generates two transactions transfering it to two different owners, without transaction ordering, it's impossible to say which of the two transactions is valid and who is the new owner. 

Data integrity ensures that all transactions included in the ledger are mutually compatible, it allows users to check only the last transfer of ownership and trust that it is preceeded by a coherent chain of title. 

For blockchains to provide these two services they have to record order and content of all transactions, in general layer 2 scaling solutions provide savings by removing either one or both of these requirements.

Payment channels remove the transaction ordering requirement by defining a linear state transition protocol instead of a branching one. Reducing possible recipients of all transactions to one subject allows all possible state transition to be mutually compatible. If I determine recipient of all my transfers ahead of time, it doesn't matter in which order they will be executed, the result will always be the same. I also don't have to worry about data integrity, because all valid messages are always compatible with all other valid messages. This means that I can do all of my transactions off-chain, consider them to be always valid (instantly final) and only commit the result on-chain in the end.

Plasma in comparison defines a protocol that allows for mutually exclusive transactions, but achieves savings by moving the task of ensuring data integrity off-chain. It still uses the main chain to establish ordering of transactions through commiting their shortened representation on-chain, but it leaves checking of data integrity to users. The data integrity is only evaluated on the main chain in case of a dispute. This approach saves work for mainchain operators, but creates additional work for users who have to keep track of at least their slice of the history of the plasma chain and make sure it is valid. Other benefit is that the history of the plasma chain can be discarded as soon as it is completely settled on the main chain. The main challenge of plasma chains is data availability and mass exits, if we abstract from these two problems plasma transactions provide similar finality as the main chain transactions with fraction of the on-chain load and are much more flexible than payment channels.

Now let's take a look at OUTPACE and see how it compares. OUTPACE is not a linear protocol, it allows for mutually exclusive transactions. It aims to solve similar problems and faces similar challenges as plasma chains, one difference being that the chain operators and transaction authors are essentially the same entitiy. In contrast with plasma chains however, it doesn't rely on main chain to provide transaction ordering, in fact it doesn't really provide users with any authoritative source for order of the transactions. This leads to possible problems with transaction finality and makes doublespending a real threat. Holding a signed transaction allows user to trigger settlement of a portion of the OUTPACE channel (not necessarily to their benefit since frontrunning is possible) but not much beyond that. The only available tool for achieving finality is withdrawing funds from the channel, but using it to finalise all transactions would negate the scaling benefits of the protocol.

So users of the protocol have to trust operators to only produce mutually compatible transactions, which is further complicated by the fact that operator is a group that decides through 2/3 supermajority (see issue #2). Developers aim to mitigate this risk in the future through introducing a staking requirement for the validators that allows them to be punished in case of misbehavior. But since theoretically there is no upper bound on the amount of damage validators can cause (because they can spend the same channel budget multiple times), it will be probably hard to determine how high the stake and punishment should be.

### 1. Rule that each state update has to be authored by validator[0] not enforced in contracts

*category: unimplemented assumption*

In protocol specification (https://github.com/AdExNetwork/adex-protocol#layer-2) it is declared that all states have to be authored by "lead validator" which is technically defined as address in the `validators` array of the `ChannelLibrary.Channel` atruct with index `0`. This requirement however isn't enforced in the state validation process implemented in the `AdExCore.channelWithdraw` function.

### 2. In the worst case only 1/3 of the validators have to maliciously collude to perform double spending attack against payment channel benefactors

*category: unrecognised implicit risk*

Since transactions are validated by at least 2/3 portion of the validators, the lowest threshold for knowing participation in a double spend attack (signing two mutually exclusive transactions) is 1/3. This threshold can be calculated as: `2x - 1` where `x` is the transaction validation quorum, or more precisely expressed as absolute number of validators: `2 * ceil(x * t) - t` where `t` is the total number of validators. So at 2 validators the double spending threshold is 2, at 3 it's 1 and at 4 it's 2 and so on.

Without the full context of additional validator staking/slashing and reputation systems, it's impossible to evaluate how problematic this property of the system is. But we feel this risk is not fully appreciated in the current documantation, which is reflected in statements like:

"Publishers have a constant guarantee that they can withdraw their latest earnings on-chain;" (https://github.com/AdExNetwork/adex-protocol#flow)

which are not strictly true.

### 3. Unnecessary restriction on who can call `channelWithdrawExpired` and `channelWithdraw` functions

*category: unnecessary complexity*

since executing `channelWithdrawExpired` and `channelWithdraw` functions is always to the benefit of the recipient, the restriction that they can be only called by the recipient doesn't seem necessary. Making them callable by anyone on anybody else's behalf would also allow `Identity` contract to be simplified.


### AdEx Identity

### 4. Allowing relayer to open channels on user's behalf poses a significant risk

Ability to open and finance OUTPACE channels on behalf of the `Identity` owners allows the routine relayer to perform powerful DoS attacks and in collusion with at least one whitelisted validator even steal funds. This is a significant risk, especially if routine relayers are intended to be provided centrally.

### 5. Same validator can be added by the relayer multiple times, reducing validator/relayer collusion treshold

At first look, routine relayer needs to collude with two whitelisted validators to be able to steal tokens from the `Identity` contract, the fact he can add the same validator multiple times however lowers this threshold to 1.

### 6. Minor griefing opportunities in IdentityFactory

Anybody can frontrun owner generated call of `deployAndFund` by either calling `deploy` or `deployAndExecute` making the owner's transaction fail. This forces the factory owner to fund the Identity contract in a different way, forcing additional transaction costs.

### 7. `WitdrawTo` authorisation should probably rank below `Transactions` authorisation

Since anybody with the `Transaction` authorisation has a complete control over the `Identity` contract, including setting authorisation levels, `WithdrawTo` is a practically redundant authorisation level (besides serving as a whitelisting device for relayer initiated withdraws). Putting it under `Transactions` authorisation would make the authorisation system more expressive.

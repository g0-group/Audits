# Audit report
## DAOStack
Repositories
- https://github.com/daostack/infra/commit/7fedc22ce3f11caf79a0ac35f3651d8231a5ae23
- https://github.com/daostack/arc/commit/8a6fbc52a23bda6677b58005ca61a0e4ace34218

## Files

All solidity files included in the repositories

## Issues
### Infra

### 1. Negative sum reputation flow

*severity: critical*

Lost reputation during pre boosted voting doesn't get fully redistributed to the winning side making the pre boosted voting a negative sum game that leads to decrease of total reputation supply.
 - lostReputation gets set to: `proposal.preBoostedVotes[losingVote] * params.votersReputationLossRatio / 100` on`line 324`, so it's the amount of burned reputation from all losing pre boosted votes
 - this then gets redistributed among winning voters on line 356 so that each gets: `(voter.reputation / (preBoostedVotes[YES] + preBoostedVotes[NO])) * lostReputation`, or in other words portion of lostReputation proportional to portion of their vote in all pre boosted cast reputation, this means that at most:
 `lostReputation * (preBoostedVotes[winningVote] / (preBoostedVotes[YES] + preBoostedVotes[NO]))` gets redistributed, instead of the whole lostReputation which would be the case if each winning voter got: `(voter.reputation / (preBoostedVotes[winningVote])) * lostReputation`,

#### fixed

Issue no longer present in https://github.com/daostack/infra/commit/ede50f61ceaaa3af5e7e4f8ab78b7c18455b2bec

It was addressed by changing the reward formula on `line 364`.

### 2. Virtual DAO stake is redeemable in real GEN tokens

*severity: critical*

When stakes put on proposal that expired in queue are being returned on `line 330`, dao bounty stake is not held back, so tokens are returned even on stake that has been created in propose function. This stake however is not a "real" stake, because no GEN tokens have been transferred to the GenesisProtocol contract to back it. This allows an attacker to create a proposal, let it expire and then steal GEN tokens from the ownership of the GenesisProtocol contract through calling redeem function.

### fixed

Issue no longer present in https://github.com/daostack/infra/commit/ede50f61ceaaa3af5e7e4f8ab78b7c18455b2bec

Was addressed by `proposals[_proposalId].stakers` array entry no longer being created in `propose` function, instead the DAO GEN reward is tracked through a new `proposals[\_proposalId].daoRedeemItsWinnings` flag and is processed on `lines 348-354` in the redeem function.

### 3. Possible underflow in redeem function formula calculating staker reward

*severity: major*

On `line 336`, if `proposal.daoBounty > (100 - proposal.expirationCallBountyPercentage)/totalStakes`, `_totalStakes` will underflow. This is caused by the inconsitent calculation of call bounty reward between `executeBoosted` and `redeem` functions.

### fixed

Issue no longer present in https://github.com/daostack/infra/commit/ede50f61ceaaa3af5e7e4f8ab78b7c18455b2bec

It was addressed by changing the call bounty reward formula on `line 330`, additional preventative measures were added in the form of using safemath library on `line 338` and an if condition on `line 337`, these doesn't seem strictly necessary, but are cheap enough that we recommend keeping them in.

### 4. executeBoosted can't be called in all valid states

*severity: major*

Due to require on `line 226`, executeBoosted bounty can be collected only on proposals that are in ProposalState.Boosted, but not on proposals that entered ProposalState.QuietEndingPeriod state.

### fixed

Issue no longer present in https://github.com/daostack/infra/commit/ede50f61ceaaa3af5e7e4f8ab78b7c18455b2bec

Was addressed through adding `|| proposal.state == ProposalState.QuietEndingPeriod` to the require on `line 224` in `executeBoosted` function.

### 5. Inconsistent calculation of executeBoosted reward across code 1/2

*severity: major*

Expiration call bounty is calculated as a portion of YES votes in executeBoosted function, but is implicitly considered to be portion of all stakes on line 336 in redeem function. This leads to more GEN tokens than has been payed out in bounty being held back during rewarding winning stakers, making staking a negative sum game.

### fixed

Issue no longer present in https://github.com/daostack/infra/commit/ede50f61ceaaa3af5e7e4f8ab78b7c18455b2bec

It was addressed by changing the call bounty reward formula on `line 330`.

### 6. Inconsistent calculation of executeBoosted reward across code 2/2

*severity: major*

If executeBoosted is succesfully called on proposal in which "NO" vote has won, the bounty is paid out, but it isn't held back in the redeem function, leading to total payout that surpasses total amount of staked tokens on the proposal.

### fixed

Issue no longer present in https://github.com/daostack/infra/commit/ede50f61ceaaa3af5e7e4f8ab78b7c18455b2bec

It was addressed by changing the reward formula on `line 342`.


### ARC

### 7. Possible collision of proposalIds in voting schemes, allows past votes to be rewritten and past voting machines to vote on behalf of future ones

*severity: minor (fragility issue, currently not directly exploitable)*

There's no mechanism preventing collision of proposalIds in voting schemes, voting machines are trusted to provide globally unique ids.
Collisions could be set-up on purpose in anticipation of proposalIds of different future voting machines, these then can be used to execute proposals registered with these voting machines.
DAO newcomers have to make sure no such collision is set-up to be able to trust the DAO will behave correctly, which might not be trivial because it involves precalculating all possible future proposalIds and comparing them with all past records in proposalsInfo array, including already executed ones.

Another related issue is that DAO can "cancel" any already approved contribution reward by using a custom voting machine to overwrite already existing record in the organizationsProposals array. Similar retroactive cancelling issue is present in GenericScheme and in VestingScheme.

### not yet addressed

### 8. Improper end time checking in Auction4Reputation.sol

*severity: minor*

In Auction4Reputation.sol

require(now <= auctionsEndTime, "bidding should be within the allowed bidding period");

should be

require(now < auctionsEndTime, "bidding should be within the allowed bidding period");

otherwise if now == auctionsEndTime, auctionId will end up being out of bounds leading to more rewards being allocated than are included in the pool

### fixed

Issue no longer present in https://github.com/daostack/infra/commit/ede50f61ceaaa3af5e7e4f8ab78b7c18455b2bec

It was addressed by changing the the code as suggested in the issue description.

### 9. Tokens in custody of VestingScheme can be stolen

*severity: critical*

Avatar and Controller contracts are incorrectly expected to follow native behaviour when `nativeToken()` and `mintTokens()` is called. 
Attacker can use non-standard contracts to claim ownership of a token vested by another user by returning an incorrect `nativeToken` value and always returning `true` when `mintTokens` is called. 
Doing this he can empty the contract of token balances in its custody.

### not yet fixed!

Issue no longer present in https://github.com/daostack/infra/commit/???

It was(n't) addressed by DAO token reward being minted in the `collect` and `cancelAgreement` function.

### 10. Approved transfers to ContributionReward can be maliciously redirected

*severity: major*

User calling `proposeContributionReward` has no assurance `nativeToken` and `controllerParams.orgNativeTokenFee` doesn't change after they submit their transaction.
A DAO can steal any token transfers user pre-approved to be transferred to the scheme by changing the return value of `nativeToken()` before proposal is processed.

### fixed

Issue no longer present in https://github.com/daostack/infra/commit/ede50f61ceaaa3af5e7e4f8ab78b7c18455b2bec

It was addressed by the proposal fee functionality being removed.

### 11. UController address squating

*severity: minor*

Reputations and tokens arrays in UController that are used in newOrganization function can be used to preemptively claim token and reputation addresses of other projects. Since addresses are generated deterministically, this could be used to block operation of DaoCreator contract.

### fixed

Issue no longer present in https://github.com/daostack/infra/commit/ede50f61ceaaa3af5e7e4f8ab78b7c18455b2bec

It was addressed by `nativeToken` and `reputation` ownership checks on `ines 98,99`. The fix introduced slight inefficiency in `DaoCreator._forgeOrg()` which could be removed by moving 

```
nativeToken.transferOwnership(address(controller));
nativeReputation.transferOwnership(address(controller));
```
above if condition on `line 199`.

### 12. UController allows organisations to claim other organisation's native token as its reputation and vice versa

*severity: critical*

If avatar provides native token address of a different organisation as a return value of `nativeReputation()` call during `newOrganization()` call or address of reputation contract of other organisation as a return value of `nativeToken()`, it will be able to mint tokens or reputation of this organisation.

### fixed

Issue no longer present in https://github.com/daostack/infra/commit/e5c68dbf4156dafdc7367e78b7ff5e872f767d28

It was addressed by unifiing registry of reputation and native token contracts in `UController` into one array called `actors`.

## Notes

There could be an effort made to improve separation of privileges. Right now all schemes can transfer unlimited amount of any tokens the dao owns, mint reputation without restrictions and burn it. In result all approved voting machines in these schemes can do the same. Privilege of burning and minting reputation can be escalated to any other privilege that a scheme that uses voting has. 
So in effect, compromise of any scheme or voting machine leads to complete compromise of the DAO rendering the permission system in the controllers largely ineffective.

## Misc

DaoCreator.sol
	line 16: permission literals could be replaced by constants to improve code readability

ExternalLocking4Reputation.sol

	line 84: why add(0x20, 0) instead of 0x20?

To prevent unexpected sideffects, all contracts in the schemes folder should be initialised before calling of any state modifying functions is allowed

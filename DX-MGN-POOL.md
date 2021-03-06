# Audit report
## DX-MGN-POOL
Repository: https://github.com/gnosis/dx-mgn-pool

Commit: 0dfe8c6df94600e5d0404547532e80e8f54262fa

## Files

- contracts/DxMgnPool.sol
- contracts/Coordinator.sol

## Issues

### 1. Inefficient tracking of participation status

Participations can only be withdrawn all at once, tracking a withdrawn status for each of them separately is inefficient.

#### fixed

60adaddf7ddd39e87ce056dcdc258386c140b839

### 2. Possible arbitrage at the expense of pool participants

There's a possible attack vector resulting from outside observers knowing limit of future result of currently running auction sooner than the contract. For simplicity let's say the contract is set up so that there's the minimum nonzero amount of auctions, just 2. First somebody puts in 1 token and gets 1 share, totalDeposit = 1, totalPoolShares = 1. Then first auction is run resulting in some amount of the secondary token bought and the second auction is started, somebody puts in enough on the buy side that it's certain that after the auction ends, totalDeposit will end up being > 1. Contract still has variable totalDeposit = 1 and totalPoolShares = 1 in its accounting, so anybody can come in and buy 1 share at the price of 1 token knowing that after current auction concludes, totalDeposit will end up being > totalPoolShares. In the end, they will get more than one token for each share they bought.

The arbitrage opportunity could be addressed by implementing a deposit queue so that all investments are realised at future not the past price. There's a way to do deposit queues with constant gas cost for the operator. An array of participationPhases could be added, which would contain structures with totalPhaseDeposit and totalPhaseShares members. On deposit, just the deposit amount for the participant is stored, index of the current participationPhase, at the same time totalPhaseDeposit is incremented for the current participationPhase by the deposit value. When auction buying deposit tokens ends, the whole totalPhaseDeposit is taken as one bulk investment and number of shares is calculated and set as a value of totalPhaseShares of the participationPhase. Then on withdrawal, participation shares are calculated as: participation.deposit * participationPhase.totalShares / participationPhase.totalDeposit

#### response

The Gnosis team is aware of this arbitrage opportunity. We decided to not increase the complexity of the contract with regards to calculating the shares of each contribution because we believe that the presented strategy still exposes the arbitrator to major volatility risk.

In the described strategy the arbitrator would still be exposed to market price fluctuations for the remainder of the pooling period (at least the last auction).

Assuming there is only one auction remaining, they can not influence its closing price unless they are willing to spend money on market manipulation (they could prevent the price from falling under the fair market price, which would benefit all participants).

Note, that all proceedings from attempted market manipulation by the buying party are shared amongst all participants in the pool. Therefore, even if the arbitrator controls a very large stake in the pool, the money they invest on the buying side would not be fully recovered.

### 3. Unnecessary storage variable initialization

L29 - L 34: initializing variables to 0 is unnecessary and results in redundant sstore.

#### fixed

60adaddf7ddd39e87ce056dcdc258386c140b839

### 4. Redundant storage reads inside loops

L82, L95: participations.length redundantly reads from state every iteration of these loops, can lift it out of their respective loops and into a function scoped variable so you only read state once

#### fixed

60adaddf7ddd39e87ce056dcdc258386c140b839

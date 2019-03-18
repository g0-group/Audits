# Audit report
## Arbitrage DutchX/Uniswap
Repository: https://github.com/gnosis/dx-mgn-pool
Commit: f015dbe8c3294bb15e4ba9b22143b554b023e047

## Files

- Arbitrage.sol

## Issues

### 1. Due to residual tokens from previous trades being mixed into new trades, unfavorable trades can be executed

In both dutchOpportunity and uniswapOpportunity, trade performance calculation is contaminated by residual performance of past trades. due to this, unfavorable trades can be wrongly evaluated as favorable and executed. The underlying problem is that only requirement for a trade is that ethereum balance of the contract stays the same or increases regardless of the change of balances of other valuable tokens that the contract controls. Since this vulnerability allows other traders to profit at the expense of the contract's owner, it will very likely be exploited.

#### fixed

846a5a2b298af2e61194171740f6e7500e6b760c

### 2. Token re-approvals can be executed conditionally

By checking if new token approval is needed before executing it, expected average gas consumption can be reduced. This applies to lines 48, 131 and 171.

#### fixed

60adaddf7ddd39e87ce056dcdc258386c140b839

### 3. Possible efficiency improvements by not fetching data prematurely

In both dutchOpportunity and uniswapOpportunity, instructions can be reordered in a way that makes calls cheaper in case of unexpected failure. Calls at lines 200 and 203 can be moved after line 212 without change in functionality, similarly call at line 151 can be moved after line 163. In general to achieve maximum efficiency even in the cases when calls throw mid-way through execution, instructions should be executed as late as possible.

#### fixed

60adaddf7ddd39e87ce056dcdc258386c140b839

### 4. We recommend using a standardized library for safe ERC20 transfers

In an effort to support faulty implementations of ERC20 standard, low level calls are used instead of high level interface calls, this is a valid apporach, but we recommend using a standardized library instead of relying on custom implementation of this functionality.

#### fixed

60adaddf7ddd39e87ce056dcdc258386c140b839

### 5. Gas limit specified on low level calls without reason

In general gas limit shouldn't be specified on external calls unless there's a specific reason to do it.

#### fixed

60adaddf7ddd39e87ce056dcdc258386c140b839

### 6. Addresses of trusted contracts can be hardcoded

Since the addresses are known at deploy time and cannot be changed afterwards, you can make them constant state variables. Constant state variables don't take up a storage slot, so would save gas there. Additionally would save gas by avoiding the overhead of setting those variables in the constructor

#### fixed

60adaddf7ddd39e87ce056dcdc258386c140b839

### 7. Unused Events

On Lines 140-141 there are Debug logs that were used for testing. Remember to remove these before deployment.

#### fixed

60adaddf7ddd39e87ce056dcdc258386c140b839

### 8. Some functions currently marked public can be marked external instead

If a public function is never called internally you can make them external. Doing so will save some gas, and is otherwise equivalent.

#### fixed

60adaddf7ddd39e87ce056dcdc258386c140b839
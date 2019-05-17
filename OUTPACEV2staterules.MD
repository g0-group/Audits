# One account one state rules

	## 1) account has to end before channel capacity

		`offset + balance <= capacity`

# Two account one state rules

	## 2) two accounts can't overlap

		`offsetA + balanceA <= offsetB || offsetA >= offsetB + balanceB`

	## 3) account has to end at the offset of the following account

		`indexB != indexA + 1 || offsetA + balanceA == offsetB`

# One account two state rules

	## 4) address of account can't change, unless original address is null

		`address1 == null || (index1 == index2 && address1 != address2)`

	## 5) balance and offset of account can only increase or stay the same
		
		`balance1 <= balance2 && offset1 <= offset2`

	## 6) end of an account (balance + offset) can increase at most by the remaining balance in the original state

		`address1 == null || (offset2 + balance2  <= offset1 + balance1 + (capacity1 - totalSpending1))`

	## 7) if account is a new one, it has to start after total spending and balance has to be smaller than remaining balance

		`address1 != null || (offset2 >= totalSpending1 && balance2 <= (capacity1 - totalSpending1))`


# Two account two state rules

	## 8) offset of an existing account after changed account has to increase by the cummulative increase in offset and balance of the changed account

		`indexA >= indexB || (offsetA2 + balanceA2) - (offsetA1 + balanceA1) <= offsetB2 - offsetB1`

	## 9) offset of a newly created account has to start after the total spending from the original state increased by a cummulative increse in offset and balance of an existing account

		`addressB1 != null || offsetB2 >= totalSpending1 + (offsetA2 + balanceA2) - (offsetA1 + balanceA1)`
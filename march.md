## [[M-11]](https://code4rena.com/reports/2022-01-trader-joe) RE-ENTERABLE CODE WHEN MAKING A DEPOSIT TO STAKE

## Impact
Note: this attack requires rJoe to relinquish control during tranfer() which under the current RocketJoeToken it does not. Thus this vulnerability is raised as medium rather than high. Although it’s not exploitable currently, it is a highly risky code pattern that should be avoided.

This vulnerability would allow the entire rJoe balance to be drained from the contract.

## Proof of Concept
The function deposit() would be vulnerable to reentrancy if rJoe relinquished control flow.

The following lines show the reward calculations in variable pending. These calculations use two state variables user.amount and user.rewardDebt. Each of these are updated after _safeRJoeTransfer().

Thus if an attacker was able to get control flow during the rJoe::tranfer() function they would be able to reenter deposit() and the value calculated for pendingwould be the same as the previous iteration hence they would again be transferred pending rJoe tokens. During the rJoe transfer the would again gain control of the execution and call deposit() again. The process could be repeated until the entire rJoe balance of the contract has been transferred to the attacker.

        if (user.amount > 0) {
            uint256 pending = (user.amount * accRJoePerShare) /
                PRECISION -
                user.rewardDebt;
            _safeRJoeTransfer(msg.sender, pending);
        }
        user.amount = user.amount + _amount;
        user.rewardDebt = (user.amount * accRJoePerShare) / PRECISION;
### Recommended Mitigation Steps
There are two possible mitigations. First is to use the openzeppelin reentrancy guard over the deposit() function which will prevent multiple deposits being made simultaneously.

The second mitigation is to follow the checks-effects-interactions pattern. This would involve updating all state variables before making any external calls.

------------------

## [[M-01]](https://code4rena.com/reports/2021-11-bootfinance/) UNCHECKED TRANSFERS

## Impact
Multiple calls to transferFrom and transfer are frequently done without checking the results. For certain ERC20 tokens, if insufficient tokens are present, no revert occurs but a result of “false” is returned. It’s important to check this. If you don’t, in this concrete case, some airdrop eligible participants could be left without their tokens. It is also a best practice to check this.

## Proof of Concept
BasicSale._processWithdrawal
AirdropDistribution.claim
InvestorDistribution.dev_rugpull

## Impact
As the trusted mainToken token is used which supposedly reverts on failed transfers, not checking the return value does not lead to any security issues.
We still recommend checking it to abide by the EIP20 standard.


## Recommended Mitigation Steps
Check the result of transferFrom and transfer. Although if this is done, the contracts will not be compatible with non standard ERC20 tokens like USDT. For that reason, I would rather recommend making use of SafeERC20 library: safeTransfer and safeTransferFrom.

------

## [[M-02]](https://code4rena.com/reports/2021-11-bootfinance/) UNCHECKED LOW-LEVEL CALLS

## Impact
The return value of these low-level calls are not checked, so if the call fails, the Ether will be locked in the contract. Setting the risk as medium as the smart contract has no function to withdraw the Ether. This Ether would remain stuck in the contract forever.

BasicSale.receive() (contracts/PublicSale.sol#148-156) ignores return value by burnAddress.call{value: msg.value}() (contracts/PublicSale.sol#154)
BasicSale.burnEtherForMember(address) (contracts/PublicSale.sol#158-166) ignores return value by burnAddress.call{value: msg.value}() (contracts/PublicSale.sol#164)

## Proof of Concept
https://github.com/code-423n4/2021-11-bootfinance/blob/main/tge/contracts/PublicSale.sol#L154
https://github.com/code-423n4/2021-11-bootfinance/blob/main/tge/contracts/PublicSale.sol#L164

## Recommended Mitigation Steps
The return value of the low-level call is not checked, so if the call fails, the Ether will be locked in the contract. If the low level is used to prevent blocking operations, consider logging failed calls.


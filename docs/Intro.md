---
sidebar_position: 1
slug: /
---

# Introduction

## Dfinity Fungible Token Standard
DFT means **Dfinity Fungible Token**.

## Overview
Token standard will help Dfinity ecological developers adopt the same standard and promote the prosperity of the Dfinity ecosystem

## Rules of Token Standard Design

ERC20 is the first fungible token standard in the blockchain world, and it has been fully verified and recognized. It is necessary to refer to the existing ERC20 standard when designing the [Dfinity Fungible Token Standard].

The design of [Dfinity Fungible Token Standard] will follow the following rules:

1. **Improving the ERC20 standard**

2. **Suitable for Dfinity**

## Improve ERC20 standard

### How to improve ERC20

ERC20 was created in the early days of Ethereum. During the development of the Eth ecosystem, the developer found that ERC20 was not perfect, so the developer designed the ERC223\ERC667\ERC777 standard to improve ERC20. We will design a better standard that combines the advantages of these standards.

1. ERC223
    - tries to solve the problem that the ERC20 transfer recipient (contract) does not support the transfer of tokens, and the transferred tokens will be lost (similar to sending tokens to a black hole address)

2. ERC667
    - adds transferAndCall which combines transfer function and call function, and solve the problems solved by ERC223

3. ERC777
    - use send function to replace transfer function
    - use the hook function to controls whether to accept transfer
    - use the operator authorization function to replace the approve function as a proxy transfer solution

### Improved standards

```
service: {
  name: () -> (text) query;
  symbol: () -> (text) query;
  decimals: () -> (nat8) query;
  totalSupply: () -> (nat) query;

  balanceOf: (owner: principal) -> (nat) query;
  allowance: (owner: principal, spender: principal) -> (nat) query;
  approve: (spender: principal, value: nat) -> (bool);
  transferFrom: (sender: principal, receiver: principal, value: nat) -> (bool);
  transfer: (receiver: principal, value: nat, args:opt vec nat8) -> (bool);
}
```

## Suitable for Dfinity

### Problems to be solved

The design of Token Standard will consider the difference between Dfinity and Ethereum fully, and clarify the problems to be solved:

1. No atomicity similar to EVM cross-contract calls

- Solution: [sagas](https://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf) is a good solution

2. No built-in event-emit support similar to EVM

- Question: 
  - How do others know that the transaction happened？
  - How to check transaction history？

- Consideration: There are two ideas (Pubsub/Notify) for question 1 on the forum

  - When the Token is transferred, Notify informs the recipient, which can fill the missing EVENT. But When the Token recipient not a canister, which means can not notify, it is necessary to support query transaction history.

  - Token does not have sufficient reason to implement Pub/Sub to release transaction information to a third party that is not the recipient

- Solution: 
  - Notify is a better way; 
  - It's necessary to support query transaction history;

3. Built-in storage support

- Question: The current storage increase to 8G, but the memory storage is still 4G

- Consideration:

  - Tx history should support query

  - Built-in storage can support Token to store more self-describing information

- Solution:

  - Auto-scaling storage solution
  - Token implements self-description

4. Reverse gas model
 - Question: How to prevent ddos attacks
 - Consideration: The call of the canister does not require the caller to pay gas fees(canister pay the gas fees)

- Solution: Any update call to canister should be charged to the caller

5. There are two different identities in Dfinity, Internet Identity (II for short) and Principal ID

- Question: Which identity to use as a token standard user？

- Consideration: Dfinity's II is an implementation of DID, and II is based on Principal ID

- Solution: It is necessary for the Token standard to be compatible with both identity, in order to meet the needs of different identity scenarios

6. No black hole address

- Question: If there is a need to destroy Token, how to deal with it?
- Consideration: Whether to implement the burn interface should be considered in the Token standard

- Solution: The burn interface does not need to be put into the standard, it can be designed in the extension interface

7. approve/transferFrom (Approve is a pairing function for TransferFrom)

- Question: There was a disagreement about whether the approval/transfer should be removed in the discussion on the forum

- Consideration:

  approve/transferFrom appears in ERC20 mainly because:

  > Using Ethereum's native ETH token, you can call the smart contract function and send ETH to the contract at the same time. This is done using payable. But because the ERC20 token itself is a smart contract, it is not possible to directly send the token to the smart contract while calling one of its functions; therefore, the ERC20 standard allows smart contracts to transfer tokens on behalf of the user-using the transferFrom() function. For this, users need to allow smart contracts to transfer these tokens on their behalf

  However, in the Dex and lending scenarios of Ethereum, Approve is often accompanied by the possibility of simultaneous operation of two tokens. Approve can avoid the repeated payment problem which transaction brought about, has a good supplementary use scenario for transfer.

- Solution: Approve/transferFrom should be retained

8. TransferAndCall vs Receiver Notify
- Question: which option is more suitable
- Consideration:
  - Notify can meet the basic notification needs. Although it cannot support better flexibility, it is sufficient to meet the transfer scenario
  - TransferAndCall provides better flexibility, but it depends on the transfer caller to fully understand the method and parameters corresponding to the call, which is not needed for most transfer scenarios
  - For security reasons, For security reasons, do not call the canister of the location inside the canister,[why?Inter-canister calls](https://www.joachim-breitner.de/blog/788-How_to_audit_an_Internet_Computer_canister)
  
- Solution: 
  - Notify is supported (cdk0.5.1, one-way notify)

9. ApproveAndCall VS TransferAndCall

- Question: We compare ApproveAndCall and TransferAndCall. ApproveAndCall and TransferAndCall are two sets of non-atomic operations, there is no difference essentially. Which one should be retained?

- Consideration: In some scenarios, when multiple Tokens need to be transferred at the same time, TransferAndCall can not meet such needs. After approval, execute transferFrom in the final call to pay multiple tokens at once
- For security reasons like Q8

- Solution: call is not supported

10. Nonce

- Question: What Is Nonce In Ethereum? every transaction has a nonce. The nonce is the number of transactions sent from a given address. Each time you send a transaction, the nonce increases by 1. There are rules about what transactions are valid, and the nonce is used to enforce some of these rules. Does Dfinity's token standard need to incorporate a nonce mechanism?

- Consideration:If the nonce mechanism is supported, it can prevent users from repeatedly submitting transactions

- Solution: use created time to implement a simple nonce mechanism

### What other functions does the Dfinity Fungible Token standard need to implement?
 
**Information self-describing**

Etherscan, MyEthereumWallet, Imtoken, TokenPocket, Dapp all have more information requirements for ERC20, such as Logo, introduction, white paper, social media, official website, contact information, etc. Each of them that needs this information needs to be maintained independently, so information appears Inconsistent. It is necessary to solve this problem through the design of <strong>[Dfinity Fungible Token Standard]</strong>

Based on the above problems and requirements,  the following draft standards are formulated:

## Standard

[DFT standard](./FungibleToken/Standards)

## References

- [1] [Dfinity Developer Center: Canister interface](https://sdk.dfinity.org/docs/interface-spec/index.html#system-api-imports)

- [2] [Dfinity Forum: thoughts-on-the-token-standard](https://forum.dfinity.org/t/thoughts-on-the-token-standard/4694)

- [3] [Toniq-Labs: ic-fungible-token](https://github.com/Toniq-Labs/ic-fungible-token)

- [4] [SuddenlyHazel: Token-Standard](https://github.com/SuddenlyHazel/token-standard/pull/1)

- [5] [Dfinance-tech: ic-token](https://github.com/dfinance-tech/ic-token)

- [6] [Plug: Token-Standard](https://github.com/Psychedelic/standards)

- [7] [Ethereum: EIPS-EIP20 & EIP667 & EIP777 & EIP1820](https://github.com/ethereum/EIPs)

- [8] [Candid](https://github.com/dfinity/candid/)

- [9] [Why are ERC20 allowances necessary?](https://kalis.me/unlimited-erc20-allowances/)

- [10] [sudograph](https://github.com/sudograph/sudograph)

- [11] [Dfinity Self Describing Standard](https://github.com/Deland-Labs/dfinity-self-describing-standard)

- [12] [How to audit an Internet Computer canister](https://www.joachim-breitner.de/blog/788-How_to_audit_an_Internet_Computer_canister)

- [13] [infinity-swap: IS20](https://github.com/infinity-swap/IS20)

- [14] [ICLighthouse: DRC20](https://github.com/iclighthouse/DRC_standards)

- [15] [Dfinity: ledger](https://github.com/dfinity/ic/tree/master/rs/rosetta-api/ledger_canister)


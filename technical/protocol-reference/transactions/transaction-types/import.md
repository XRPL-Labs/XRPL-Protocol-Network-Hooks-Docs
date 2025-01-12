---
description: >-
  Import is a new transaction which accepts an XPOP from the XRPL Mainnet
  chain (network_id=0) or Testnet (network_id=1) and provides account synchronisation.
---

# Import

\[[Source](https://github.com/Xahau/xahaud/blob/dev/src/ripple/app/tx/impl/URIToken.cpp)]

_(Added by the \[Import amendment]\[].)_

### Example

```json
{
  "TransactionType": "Import",
  "Sequence": 0,
  "Fee": "0",
  "Account": "rUn84CUYbNjRoTQ6mSW7BVJPSVJNLb1QLo",
  "Blob" : "DEADBEEF"
}
```

### Fields

| Field         | JSON Type | \[Internal Type]\[] | Description                                          |
| ------------- | --------- | ------------------- | ---------------------------------------------------- |
| `Blob`        | String    | Blob                | Hex value representing an XPOP                       |
| `Issuer`      | String    | AccountID           | (Optional) Address that can be used inside the Hook. |
| `Destination` | String    | AccountID           | (Optional) Address that can be used inside the Hook. |

### Error Cases

* If the account is Non Activated then the `Sequence` must be 0 and the `Fee` must also be 0
* If the account is Activated then the `Sequence` and the `Fee` are calculated using the standard method.
* If the `Issuer` field is present then the `Fee` must be calculated using the standard method.

### Notes

_It is recommended that if you use a `SignersList` or `RegularKey` to sign your transactions that you key your accounts **FIRST** before attempting to B2M_ XAH_._

* If the inner (xpop) transaction is `AccountSet` the mainet existing flags will be transfered to the new network.
* If the inner (xpop) transaction is `SetRegularKey` with the `RegularKey` field omitted or empty, and a signers list does not exist for the account then the `lsfDisableMaster` flag will be set on the account.
* If the inner (xpop) transaction is `SetRegularKey` then the `lsfPasswordSpent` flag will be set on the account.
* TicketSequence is not available on `Import`

### Importing for the Issuer

For issuers, there are additional steps to follow before their asset holders can import transactions.

Firstly, issuers need to install a hook. There are two options for this: `B2MNFToken` or `B2MPayment`.

#### B2MNFToken

A `NFTokenBurn` transaction on mainnet will result in a `URITokenMint` transaction on the network.

> c hook: https://example.com

#### B2MPayment

A `Payment` transaction on mainnet to the Issuer will result in a `Payment` transaction from the Issuer on the network.

> c hook: https://example.com

_Please note that the process of importing for the issuer involves specific transaction types and requires careful configuration. Always ensure that the hooks are correctly set up and that the transactions are valid for the intended operations._

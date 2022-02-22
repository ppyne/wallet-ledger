# Ledger wallet access for Stellar

Provide a browserifyed Ledger wallet access for Stellar

- https://github.com/LedgerHQ/ledgerjs
- https://github.com/LedgerHQ/ledgerjs/tree/master/packages/hw-app-str

## Warning

Access to USB devices can only be done over SSL (HTTPS) and `Transport.create()` must be triggered by a button.

## Prerequisite

```sh
npm install -g browserify
```

## Install

```sh
cd wallet-ledger
npm install
```

## Examples

```html
    <script src="https://cdnjs.cloudflare.com/ajax/libs/stellar-sdk/10.0.1/stellar-sdk.min.js"
        integrity="sha512-pDFM6HQlPpmbVLOENLgRZsl9V4HL5Lu6FOqiZbPWmLC/698C5gXG7+OZCOEQJhfggwE/cEGy3FwPmKPRBXQFWg=="
        crossorigin="anonymous"
        referrerpolicy="no-referrer"></script>
    <script src="walletledger.js"></script>
```

```javascript
const getStrAppVersion = async () => {
    const transport = await Transport.create();
    const str = new Str(transport);
    const result = await str.getAppConfiguration();
    return result.version;
}
getStrAppVersion().then(v => console.log(v));

const getStrPublicKey = async () => {
  const transport = await Transport.create();
  const str = new Str(transport);
  const result = await str.getPublicKey("44'/148'/0'");
  return result.publicKey;
};
let publicKey;
getStrPublicKey().then(pk => {
    console.log(pk);
    publicKey = pk;
});

const signStrTransaction = async (publicKey) => {
  const transaction = new StellarSdk.TransactionBuilder({accountId: () => publicKey, sequenceNumber: () => '1234', incrementSequenceNumber: () => null})
    .addOperation(StellarSdk.Operation.createAccount({
       source: publicKey,
       destination: 'GBLYVYCCCRYTZTWTWGOMJYKEGQMTH2U3X4R4NUI7CUGIGEJEKYD5S5OJ', // SATIS5GR33FXKM7HVWZ2UQO33GM66TVORZUEF2HPUQ3J7K634CTOAWQ7
       startingBalance: '11.331',
    }))
    .build();
  const transport = await Transport.create();
  const str = new Str(transport);
  const result = await str.signTransaction("44'/148'/0'", transaction.signatureBase());
  
  // add signature to transaction
  const keyPair = StellarSdk.Keypair.fromPublicKey(publicKey);
  const hint = keyPair.signatureHint();
  const decorated = new StellarSdk.xdr.DecoratedSignature({hint: hint, signature: result.signature});
  transaction.signatures.push(decorated);
  
  return transaction;
}
signStrTransaction(publicKey).then(transaction => console.log(transaction.toEnvelope().toXDR().toString('base64')));
```



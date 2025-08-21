# Vote as and SPO

### Step 1: Hot environment queries

#### 1st Query
Query the governance state where you will get the governance action ID you need to build your offline vote file
```
cardano-cli conway query gov-state \
--mainnet \
--out-file gov-state.json
```

#### 2nd Query
Query the UTXO of the payment address you plan to use for the vote transaction (ideally one for which you have the signing keys stored in your cold environment).
```
cardano-cli conway query utxo \
--mainnet \
--address <YOUR WALLET ADDRESS> \
--out-file wallet-utxo.json
```

### Step 2: Transfer Both Files to Cold Environment
Transfer these two files to your cold environment to securely build the vote file, construct the transaction, and perform the signing process offline. 
This minimizes exposure to potential online threats during the transaction.

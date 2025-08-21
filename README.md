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

#### Locate the UpdateCommittee Proposal and Extract the Governance Action ID
Scroll to the bottom of the `gov-state.json`file in your cold environment, locate the proposal labeled UpdateCommittee, 
and copy the governance action ID.
<img width="2579" height="2000" alt="SPOvote" src="https://github.com/user-attachments/assets/1ba8e914-2872-4a25-abc3-c98187a6ac91" />

#### Build the vote file
Build your vote file with your chosen vote and optional rationale. If including a rationale feels too complex, you can omit the `--anchor-url` and `--anchor-data-hash` parameters.
```
cardano-cli conway governance vote create \
--yes \
--governance-action-tx-id 47a0e7a4f9383b1afc2192b23b41824d65ac978d7741aca61fc1fa16833d1111 \
--governance-action-index 0 \
--anchor-url <YOUR RATIONAL URL LINK> \  #Providing a rationale is optional
--anchor-data-hash <YOUR RATIONAL HASH> \ #Providing a rationale is optional
--cold-verification-key-file cold.vkey \
--out-file action.vote
```

#### Retrieve UTXO from `wallet-utxo.json` for Transaction Input
Open your `wallet-utxo.json` file in your cold environment and identify a suitable UTXO to use as the transaction input. You'll need the transaction hash and index (txHash#index) for the next steps.
<img width="2788" height="956" alt="SPOvote2" src="https://github.com/user-attachments/assets/45912139-9310-4567-a372-e696604033ce" />

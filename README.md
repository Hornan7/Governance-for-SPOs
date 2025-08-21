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

#### 3rd Query
Query the protocol parameters from your online environment so you can use them later to calculate the transaction fee in your cold environment.
```
cardano-cli conway query protocol-parameters \
--mainnet \
--out-file pparams.json
```

### Step 2: Transfer Both Files to Cold Environment
Transfer these three files to your cold environment to securely build the vote file, construct the transaction, and perform the signing process offline. 
This minimizes exposure to potential online threats during the transaction.

#### Locate the UpdateCommittee Proposal and Extract the Governance Action ID
Scroll to the bottom of the `gov-state.json`file in your cold environment, locate the proposal labeled UpdateCommittee, 
and copy the governance action ID.
<img width="2579" height="2000" alt="SPOvote" src="https://github.com/user-attachments/assets/1ba8e914-2872-4a25-abc3-c98187a6ac91" />

### Step 3: Build the vote file
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

### Step 4: Build Transaction Draft
We build a draft transaction first because the exact fee isn't known yet. To calculate the correct fee, we need to estimate it based on a nearly complete transaction. 
By using available inputs to create a draft, we can then calculate the fee and adjust the final transaction outputs accordingly.
```
cardano-cli conway transaction build-raw \
--tx-in <YOUR WALLET UTXO> \
--tx-out <YOUR WALLET ADDRESS>+0 \
--vote-file action.vote \
--fee 0 \
--out-file tx.raw
```

### Step 5: Calculate the transaction fee
Provide the protocol parameters file `pparams.json` and set `--witness-count` to the number of keys you'll use to sign (your pool cold key and wallet payment key); set both `--tx-in-count` and `--tx-out-count` to `1`, since you're using one input and sending the change back to yourself as a single output.
```
cardano-cli conway transaction calculate-min-fee \
--tx-body-file tx.raw \
--protocol-params-file pparams.json \
--witness-count 2 \
--mainnet \
--tx-in-count 1 \
--tx-out-count 1
```
You should get an output like this:

<img width="188" height="63" alt="image" src="https://github.com/user-attachments/assets/4127d70c-7094-446b-9416-57209149532d" />

### Step 6: Build the Final Transaction
Now that you have the transaction fee, determine the amount of lovelace to include in your output by subtracting the fee from the total value of the wallet UTXO you're spending. Use this result as the output amount in your final transaction body file.
```
expr 86843345 - 171529
```

```
cardano-cli conway transaction build-raw \
--tx-in 207b226e110e13bb18b119fcd313520e0fcd060b2bc9fb9a5e5bc6e94ab10f3b#0 \
--tx-out addr1vyae679fqna7yal7qhw6trxdd06k78ynt70qejyyw96ev3qz0m3hh+86671816 \
--vote-file action.vote \
--fee 171529 \
--out-file tx.raw
```

### Step 7: Sign the transaction
Sign your transaction body file using both `cold.skey` and `payment.skey`
```
cardano-cli conway transaction sign \
--tx-body-file tx.raw \
--mainnet \
--signing-key-file payment.skey \
--signing-key-file cold.skey \
--out-file tx.signed
```

### Step 8: Move the Signed Transaction to the Hot Environment
Move the signed transaction file to your hot (online) environment to submit it to the blockchain. This file does not contain any sensitive private key data — it only includes the transaction details and cryptographic witnesses required for validation, making it safe to transfer across environments.
```
cardano-cli conway transaction submit \
--mainnet \
--tx-file tx.signed
```
And voilà, congratulations! You’ve successfully cast your vote as a Stake Pool Operator in our awesome governance system

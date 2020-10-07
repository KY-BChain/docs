

## **Interacting with the Agentland Test-net**

The agent land test-net is Fetch.ai's latest test-net for running agent applications. You can access the block-explorer at this URL:

[https://explore-agent-land.fetch.ai/](https://explore-agent-land.fetch.ai/)

The landing page has panels showing the block-height, block time, the list of validators that are currently online and total voting power. The graph in the bottom right-hand corner shows block production time for the blocks that were generated most recently along with validator numbers. (Validators operated by Fetch are named after secret agents).


### **Getting Tokens**

The get funds button can be used to obtain tokens. There are four separate ways to obtain a Fetch address for receiving tokens:

1. Through the block explorer via a[ Ledger](https://www.ledger.com/) nano device. This method also provides access to token delegations to validators and other chain governance functions.

2. Fetch.ai's[ Chrome](https://www.google.com/intl/en_uk/chrome/) browser[ extension](https://chrome.google.com/webstore/detail/fetchai-cosmos-web-wallet/hadajeigbmmhfgeomldfmgfddgeggnmj?hl=en&authuser=1), which is suitable for token transfers.

3. The _fetchcli_ command-line interface for access to all token transfer and governance mechanisms.

4. The agent framework interface.

The first two approaches are appropriate for all users, and are compatible with Ledger hardware wallets, while the latter two approaches are recommended for developers.

### **Hardware Wallets**

We recommend hardware wallets as a solution for managing private keys. The Fetch v2.0 ledger is compatible with Ledger nano devices. To use your ledger nano, your first step is to[ set-up](https://www.ledger.com/start/) your wallet by creating a PIN and passphrase, which must be stored securely to enable recovery if the device is lost or damaged. You should then connect your device to your PC and update the firmware to the latest version using the[ Ledger Live](https://www.ledger.com/ledger-live/download) application. The final step involves installing the Cosmos application using the software manager (_Manager > Cosmos > Install_). Interacting with the Fetch blockchain involves connecting your device, selecting the Cosmos application and accessing pages and extensions using a Chrome web browser.


### **1. Block Explorer**

Return to the block explorer[ landing page](https://explore-agent-land.fetch.ai/) and click on the key button in the top right corner. You'll then be prompted to _Sign in With Ledger_. You must accept this request on your ledger nano device. After completing this process, the key button will be replaced by a person icon with a link to your personal address page, which keeps track of the activity that you have performed on the test-net. This includes the funds that you currently have available, which should be zero at this stage.


#### Getting Test-Net Tokens from the Faucet

You can obtain tokens for your account by copying the address and pasting it into the token faucet; return to the[ main page](https://explore-agent-land.fetch.ai/), press the _Get Funds_ button and paste your address in the pop-up. Afterwards you can return to your address page (via the person icon) and should observe that you have been allocated 1 TESTFET.


#### Transferring Tokens to another Address

After receiving tokens, you can send these to another address using the purple _Transfer_ button on your address page.  This will trigger a pop-up that prompts you to specify the destination address and the amount you wish to transfer. After filling in this information, you will be asked to sign the transaction using your ledger nano. The confirmation that the transaction has been broadcast gives two links that can be used to check that the transaction has been executed on the blockchain using either the transaction hash or your account page. (Tip: the transaction format includes a _memo_ field that can be used to check the transaction information on the ledger nano display.).


#### Delegating Test-Net Tokens to a Validator

You can delegate your test-net tokens to a validator who is operating the network by clicking on the[ validators tab](https://explore-agent-land.fetch.ai/validators), and selecting one of the validators that you wish to delegate tokens to. In the _Voting Power_ panel there are options to _Delegate_, _Redelegate_ and _Undelegate_ tokens. The delegation of tokens to a validator provides you with a reward for helping to secure the network. It is also possible to delegate your tokens to a different validator using _Redelegate_ request. You can return any bonded tokens to your address by submitting an _Undelegate_ request, which will trigger the tokens to be returned after 21 days have elapsed. The rewards that you receive from delegating tokens to a validator are shown in the account page. These can be sent to your address by selecting the _Withdraw_ button.

## **2. Chrome Browser Extension**

TBC

## 3. _Fetchcli_ command-line client

TBC

## 4. Agent Framework

In this section, we look at some Python that you can use to create addresses, fund them, and submit transactions to the network. It's super-simple to do:

```python
from aea.crypto.fetchai import FetchAICrypto, FetchAIApi, FetchAIFaucetApi

# Get a new address:
fetch_crypto = FetchAICrypto()
address = fetch_crypto

# This is relatively slow, so if you need to stock up a number of
# addresses, it’s best to get it ONCE, then distribute yourself:
# for rapid tests:
FetchAIFaucetApi().get_wealth(address)
balance = FetchAIApi().get_balance(address)
print(f”Our address {address} has a balance of {balance}”)
```

As you can see, it’s super-easy to use. The above works in standalone code, and is supported in the agent framework. Here is a code fragment that shows the construction, signing and submission of a transaction:

```python
from aea.crypto.fetchai import FetchAICrypto, FetchAIApi, FetchAIFaucetApi

# ... code, etc...

def test_construct_sign_and_submit_transfer_transaction():
    """Test the construction, signing and submitting of a transfer transaction."""
    account = FetchAICrypto()
    balance = get_wealth(account.address)
    assert balance > 0, "Failed to fund account."
    fc2 = FetchAICrypto()
    fetchai_api = FetchAIApi(**FETCHAI_TESTNET_CONFIG)
    amount = 10000
    assert amount < balance, "Not enough funds."
    transfer_transaction = fetchai_api.get_transfer_transaction(
        sender_address=account.address,
        destination_address=fc2.address,
        amount=amount,
        tx_fee=1000,
        tx_nonce="something",
    )
    assert (
        isinstance(transfer_transaction, dict) and len(transfer_transaction) == 6
    ), "Incorrect transfer_transaction constructed."
    signed_transaction = account.sign_transaction(transfer_transaction)
    assert (
        isinstance(signed_transaction, dict)
        and len(signed_transaction["tx"]) == 4
        and isinstance(signed_transaction["tx"]["signatures"], list)
    ), "Incorrect signed_transaction constructed."
    transaction_digest = fetchai_api.send_signed_transaction(signed_transaction)
    assert transaction_digest is not None, "Failed to submit transfer transaction!"

		# Now let's wait around for a while for this transaction to go through"
    not_settled = True
    elapsed_time = 0
    while not_settled and elapsed_time < 20:
        elapsed_time += 1
        time.sleep(2)
        transaction_receipt = fetchai_api.get_transaction_receipt(transaction_digest)
        if transaction_receipt is None:
            continue
        is_settled = fetchai_api.is_transaction_settled(transaction_receipt)
        not_settled = not is_settled
    assert transaction_receipt is not None, "Failed to retrieve transaction receipt."
    assert is_settled, "Failed to verify tx!"
    tx = fetchai_api.get_transaction(transaction_digest)
    is_valid = fetchai_api.is_transaction_valid(
        tx, fc2.address, account.address, "", amount
    )
    assert is_valid, "Failed to settle tx correctly!"
    assert tx == transaction_receipt, "Should be same!"
```


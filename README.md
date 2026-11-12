# chubbicorns-nft-metadata

Metadata and artwork for all 250 Chubbicorns, recovered after `app.chubbiverse.com` went offline.

Contract: [`0xb072114151f32D85223aE7B00Ac0528d1F56aa6E`](https://etherscan.io/address/0xb072114151f32D85223aE7B00Ac0528d1F56aa6E)

Hosted at `https://metadata.chubbicorns.com`.

## Contents

- `metadata/<id>.json`: name, description, image and traits for Chubbicorn 1–250
- `images/<id>.png`: original 2000×2000 artwork, linked from each metadata `image` field as `https://metadata.chubbicorns.com/images/<id>.png`
- `vercel.json`: serves `/<id>` (and the legacy `/api/meta/chubbicorn/<id>`) from `metadata/<id>.json`

Traits and images come from the original `chubbiverse/chubbiverse-web` source. They were checked against Alchemy's cached copies of the old API.

## Hosting

1. Deploy this repo to Vercel as a static site (no build step).
2. Add `metadata.chubbicorns.com` as the project's domain.
3. In the DNS for `chubbicorns.com`, add a `CNAME` record for `metadata` pointing to the target Vercel shows under the project's **Settings → Domains**.
4. Check that the domain serves metadata:

   ```sh
   curl https://metadata.chubbicorns.com/1
   ```

## Pointing the contract at the new metadata URL

The contract still returns the old, offline URL:

```
baseTokenURI() → https://app.chubbiverse.com/api/meta/chubbicorn/
tokenURI(1)    → https://app.chubbiverse.com/api/meta/chubbicorn/1
```

`tokenURI(id)` is `baseTokenURI` with the token ID appended, so switching the base URI updates all 250 tokens at once.

Contract details:

- Proxy (the address to call): [`0xb072114151f32D85223aE7B00Ac0528d1F56aa6E`](https://etherscan.io/address/0xb072114151f32D85223aE7B00Ac0528d1F56aa6E), an EIP-1967 `TransparentUpgradeableProxy`
- Implementation: [`0x5EeD8Ef173c86AA5b23E47B5EEFa3E1997912890`](https://etherscan.io/address/0x5EeD8Ef173c86AA5b23E47B5EEFa3E1997912890#code) (verified source, `Chubbicorns`)
- Owner: [`0x55256f0E4f0e6e503A38E6e31b3b44904cd8D78f`](https://etherscan.io/address/0x55256f0E4f0e6e503A38E6e31b3b44904cd8D78f). `setBaseTokenURI` is `onlyOwner`, so other wallets revert with `Ownable: caller is not the owner`.

Do this only after `https://metadata.chubbicorns.com/1` returns JSON. Otherwise the tokens will point to a URL that doesn't resolve.

### Using Etherscan

1. Open the contract's **[Write as Proxy](https://etherscan.io/address/0xb072114151f32D85223aE7B00Ac0528d1F56aa6E#writeProxyContract)** tab. Use **Write as Proxy**, not **Write Contract**, because the function lives on the implementation.
2. Click **Connect to Web3** and connect the owner wallet (`0x55256f0E…D78f`).
3. Open `setBaseTokenURI` and enter:

   ```
   https://metadata.chubbicorns.com/
   ```

   Keep the trailing slash. Without it, token 1 resolves to `https://metadata.chubbicorns.com1`.
4. Click **Write** and confirm the transaction in your wallet. It uses about 48k gas.
5. When it confirms, open **[Read as Proxy](https://etherscan.io/address/0xb072114151f32D85223aE7B00Ac0528d1F56aa6E#readProxyContract)** and check:
   - `baseTokenURI` returns `https://metadata.chubbicorns.com/`
   - `tokenURI(1)` returns `https://metadata.chubbicorns.com/1`

### Using Foundry (`cast`)

```sh
# Send from the owner wallet (use --ledger, --trezor, or --interactive to pass the key)
cast send 0xb072114151f32D85223aE7B00Ac0528d1F56aa6E \
  'setBaseTokenURI(string)' 'https://metadata.chubbicorns.com/' \
  --rpc-url https://ethereum-rpc.publicnode.com --ledger

# Verify
cast call 0xb072114151f32D85223aE7B00Ac0528d1F56aa6E 'tokenURI(uint256)(string)' 1 \
  --rpc-url https://ethereum-rpc.publicnode.com
```

### Refreshing marketplaces

The contract emits no metadata-update event, so marketplaces keep cached data until you refresh them. On OpenSea, open a Chubbicorn and click **Refresh metadata** (⋯ menu). To refresh all 250, call OpenSea's refresh endpoint for each token (requires an API key):

```sh
for id in $(seq 1 250); do
  curl -s -X POST -H "X-API-KEY: $OPENSEA_API_KEY" \
    "https://api.opensea.io/api/v2/chain/ethereum/contract/0xb072114151f32D85223aE7B00Ac0528d1F56aa6E/nfts/$id/refresh"
  sleep 0.5
done
```

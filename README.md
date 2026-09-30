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
3. From the contract owner (`0x55256f0e4f0e6e503a38e6e31b3b44904cd8d78f`), call `setBaseTokenURI("https://metadata.chubbicorns.com/")`.
   `tokenURI(1)` then returns `https://metadata.chubbicorns.com/1`.
4. Refresh metadata on OpenSea so marketplaces pick up the change.

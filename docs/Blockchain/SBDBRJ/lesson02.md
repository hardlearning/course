# PROJECT - Airdropping

## Environment set-up

```
mkdir airdrop-project
cd airdrop-project
npm init -y
npm install --save @solana/web3.js
```

## Create your own wallet

- index.js

```javascript
const {
    Connection,
    PublicKey,
    clusterApiUrl,
    Keypair,
    LAMPORTS_PER_SOL
} = require("@solana/web3.js")

const wallet = new Keypair()
```
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

// create wallet
const wallet = new Keypair()

// retrieve your wallet
const publicKey = new PublicKey(wallet._keypair.publicKey)
const secretKey = wallet._keypair.secretKey

// console.log(wallet._keypair.publicKey)
// console.log(secretKey)

const getWalletBalance = async() => {
    try {
        const connection = new Connection(clusterApiUrl('devnet'), 'confirmed')
        const walletBalance = await connection.getBalance(publicKey)
        console.log('Wallet balance is ${walletBalance}')
    } catch (err) {
        console.error(err)
    }
}

const airDropSol = async() => {
    try {
        const connection = new Connection(clusterApiUrl('devnet'), 'confirmed')
        const fromAirDropSignature = await connection.requestAirDrop(publicKey, 2 * LAMPORTS_PER_SOL)
        await connection.confirmTransaction(fromAirDropSignature)
    } catch(err) {
        console.error(err)
    }
}

const main = async() => {
    await getWalletBalance()
    await airDropSol()
    await getWalletBalance()
}
main()
```

```
node index.js
```
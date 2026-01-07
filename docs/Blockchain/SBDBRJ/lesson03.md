# DEFI PROJECT - Launch your own cryptocurrency

## Install the Solana Program Library

```
cargo install spl-token-cli
```

## Create your own wallet and check on Solana Explorer

```
solana-keygen new
solana-keygen new --force
solana-keygen pubkey
solana balance --url devnet
solana airdrop 2 <public key> --url devnet
```

[Solana Explorer](https://explorer.solana.com/)

## Create a token

```
# generate token address
spl-token create-token --url devnet
```

## Mint your token

```
# generate token account address
spl-token create-account <token address> --url devnet
spl-token balance <token address> --url devnet
spl-token mint <token address> 1000 --url devnet
```

## Limit the total supply of your token and burn your token

```
# total supplied tokens
spl-token supply <token address> --url devnet
# disable mint token
spl-token authorize <token address> mint --disable --url devnet
# now cannot mint token
spl-token mint <token address> 1000 --url devnet
# burn 100 tokens
spl-token burn <token account address> 100 --url devnet
```

## Send your token to your friends with the Phantom wallet

```
spl-token transfer <token address> 150 <friend's wallet address> --url devnet
spl-token transfer <token address> 150 <friend's wallet address> --url devnet --allow-unfunded-recipient --fund-recipient
```
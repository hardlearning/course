# Solana SPL Tokens INTRO (2025)

## Create a Solana Token with Image with Code (Typescript)

```
git clone https://github.com/serpentacademy/create-Solana-Token.git
cd create-Solana-Token
npm install
```

1. 修改src/1createPrivateKeywithVanity.ts

```typescript
import { Keypair } from '@solana/web3.js';
import bs58 from 'bs58';
import fs from 'fs';

console.log('Starting search for vanity public key ending with "tokn"...');

while (true) {
  const keypair = Keypair.generate();
  const secretKeyBase58 = bs58.encode(keypair.secretKey);
  let pubKey = keypair.publicKey.toBase58()
  //console.log(pubKey)
  if (pubKey.endsWith('tkn')) {
    const secretKeyArray = Array.from(keypair.secretKey);
    fs.writeFileSync('wallet.json', JSON.stringify(secretKeyArray));
    console.log('Found and saved private key to pumpprivate.json');
    console.log('Base58 encoded: ' + secretKeyBase58);
    break;
  }
}
```

```
npx ts-node src/1createPrivateKeywithVanity.ts
# 查看生成的公钥
solana-keygen pubkey wallet.json
```

2. src/2airdropAndDecode.ts

```
# 查看solana网络配置，确保当前使用的是devnet
solana config get
# 切换到devnet
solana config set --url devnet
# 转入sol
npx ts-node src/2airdropAndDecode.ts
# 查看sol余额
solana balance DURDcERHDdNqQV28CN2X25qM7wzEchZFhKqZdABtEtkn
```

3. src/3findaddressmultithread.ts

```
npx ts-node src/3findaddressmultithread.ts
solana-keygen pubkey vanityToken.json
```

4. src/2MintToken.ts

```
npx ts-node src/2MintToken.ts
npx ts-node src/3_4UpdateMetadataToken.ts
npx ts-node src/5mintMore.ts
npx ts-node src/14frozeauthorityToken.ts
npx ts-node src/15RevokeMintAuthority.ts
```

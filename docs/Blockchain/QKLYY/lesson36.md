# 36 Anchor本地开发与交互

## 创建Anchor工程

anchor安装：https://www.anchor-lang.com/docs/installation

```
# 创建工程
anchor init <project-name>
# 在工程下新建program
anchor new <program-name>
```

## Anchor命令

```
# 编译项目
anchor build
# 运行测试
anchor test
# 部署，准备feePay账号、RPC
anchor deploy
# 更新declare_id
anchor keys sync
```

## 部署程序

在Anchor.toml中配置provider

```
[provider]
cluster = "localnet"
wallet = "~/.config/solana/id.json"
```

使用配置provider进行部署

```
anchor deploy [-p <program_name>]
```

## 测试

Anchor项目通常两套测试：Rust单元测试 + Anchor集成测试

## 单元测试

在lib.rs里编写#[cfg(test)]，在程序的crate目录下，运行`cargo test`

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_favorites_struct_creation() {
        let favorites = Favorites {
            number: 42,
            color: "blue".to_string(),
        };
        assert_eq!(favorites.number, 42);
        assert_eq!(favorites.color, "blue");
    }
}
```

## 集成测试

集成测试用例：Anchor工程使用Mocha测试框架和Chai断言库

使用anchor client库（@coral-xyz/anchor）与Solana程序交互

[代码地址](https://github.com/lbc-team/hello_solana/blob/main/anchor_favorites/tests/anchor_favorites.ts)

```typescript
import * as anchor from "@coral-xyz/anchor";
import { Program } from "@coral-xyz/anchor";
import { Favorites } from "../target/types/favorites";
import { BN } from "@coral-xyz/anchor";
import { expect } from "chai";

async function generateUserAndAirdropSol() {

      // 创建用户 keypair（需要作为 signer）
      const user = anchor.web3.Keypair.generate();
    
      // 给用户账户空投 SOL
      const connection = anchor.getProvider().connection;
      const { blockhash, lastValidBlockHeight } = await connection.getLatestBlockhash();
      
      const airdropSignature = await connection.requestAirdrop(
        user.publicKey,
        2 * anchor.web3.LAMPORTS_PER_SOL // 空投 2 SOL
      );
      
      await connection.confirmTransaction({
        signature: airdropSignature,
        blockhash,
        lastValidBlockHeight,
      });

      return user;
}

// describe用于创建一个测试组
// describe.only只运行这个测试组
describe.only("anchor_favorites", () => {
  // 读取 Anchor.toml 配置的 cluster 与 wallet 作为 provider 
  anchor.setProvider(anchor.AnchorProvider.env());
  const program = anchor.workspace.Favorites as Program<Favorites>;

  // it定义具体的测试用例
  it("favorites!", async () => {
    // 随机生成用户并空投 SOL
    // const user = await generateUserAndAirdropSol();
    
    // 使用 Anchor.toml 中配置的钱包
    const provider = anchor.getProvider();
    const user = (provider.wallet as anchor.Wallet).payer;
    
    // 计算 PDA（根据 lib.rs 中的 seeds）
    const [favoritesPda] = anchor.web3.PublicKey.findProgramAddressSync(
      [Buffer.from("favorites"), 
      user.publicKey.toBuffer()],
      program.programId
    );

    // 调用 setFavorites 方法
    const tx = await program.methods
      .setFavorites(new BN(42), "blue")
      .accounts({
        user: user.publicKey,
        favorites: favoritesPda,
        systemProgram: anchor.web3.SystemProgram.programId,
      })
      .signers([user])  // 添加 user 作为 signer
      .rpc();
      
    console.log("Your transaction signature", tx);
    
    // 调用 program.account.specificAccountType.fetch 获取某个账户数据
    // 如果获取所有账户数据，则使用 program.account.specificAccountType.all 方法
    const favoritesAccount = await program.account.favorites.fetch(favoritesPda);
    console.log("Favorites account:", favoritesAccount);
    console.log("Number:", favoritesAccount.number.toString());
    console.log("Color:", favoritesAccount.color);

    // 断言验证账户数据
    expect(favoritesAccount.number.toString()).to.equal("42");
    expect(favoritesAccount.color).to.equal("blue");
  });
});
```

在Anchor工程的tests目录下编写测试，然后运行

```
anchor test
# 跳过启动本地服务进行测试
anchor test --skip-local-validator
```

## Solana Client Library

RPC接口: https://solana.com/zh/docs/rpc/http/getbalance

metaplex-umi: https://umi.typedoc.metaplex.com/

## 测试手段 - 打印日志

Solana支持使用msg!()和emit!()在链上打印日志
- msg! 更轻量、适合调试 , 适合打印调试、状态提示， 适合纯字符串
- emit! 是结构化(序列化和编码)、便于前端工具解析数据

### 打印日志

[代码地址](https://github.com/lbc-team/hello_solana/tree/main/anchor_favorites/programs/emit_log)

```rust
pub mod emit_log {
    use super::*;
    pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
        msg!("Program ID: {} will emit log", ctx.program_id);
        emit!(MyEvent { value: 12 });
        emit!(MySecondEvent { value: 3, message: "hello world".to_string() });
        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize {}

#[event]
pub struct MyEvent {
    pub value: u64,
}

#[event]
pub struct MySecondEvent {
    pub value: u64,
    pub message: String,
}
```

### 获取日志

使用Anchor内置EventParser解码emit日志

[代码地址](hello_solana/blob/main/anchor_favorites/tests/emit_log.ts)

```typescript
const txInfo = await connection.getParsedTransaction(sig);
console.log(txInfo.meta.logMessages);
for (const log of txInfo.meta.logMessages) {
    const prefix = "Program data: ";
    if (log.startsWith(prefix)) {
        const base64 = log.slice(prefix.length);
        const event = program.coder.events.decode(base64);
        if (event) {
            console.log("Anchor Event:", event);
        }
    }
}
```

监听程序的日志:

```
solana logs <programID> -u http://127.0.0.1:8899
```

## 交互

### 创建交互工程

1. 创建TypeScript工程：pnpm init + 安装typeScript及 TypeScript执行器（ts-node或esrun直接运行.ts）
2. 从anchor工程拷贝idl/types到交互的代码工程
3. 安装@solana/web3.js和@coral-xyz/anchor，anchor库当前只和web3.js v1版本兼容

参考代码：https://github.com/lbc-team/hello_solana/tree/main/backend

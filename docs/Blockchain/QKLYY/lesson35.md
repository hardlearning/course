# 35 第一个Solana程序

## 账号-创建

```
# 生成默认钱包，keypair文件保存在~/.config/solana/id.json
solana-keygen new
# 生成keypair文件，可直接导入的Phantom等钱包
solana-keygen new --outfile my.json
# 生成以abc开头的靓号
solana-keygen grind --starts-with abc:1
# 查看默认账号的地址
solana address
# 查看指定账号的地址
solana address -k my.json
```

Solana Cli文档：https://docs.anza.xyz/cli/usage#usage

## 账号-领水

Devnet领水：https://faucet.solana.com/

本地测试网：

```
# 启动本地测试网，默认rpc url: http://127.0.0.1:8899
# 注意：WSL中这个命令不能在/mnt目录下执行，必须在家目录下
solana-test-validator
# 从本地网络领水
solana airdrop 5 -u http://127.0.0.1:8899
solana airdrop 5 <address> -u http://127.0.0.1:8899
```

## 账户-获取余额

```
# 先查看网络设置
solana config get
# 设置网络
solana config set --url devnet
# 查看余额
solana balance
solana balance <address>
solana balance <address> -u http://127.0.0.1:8899
solana balance —keypair my.json
```

## SOLANA PLAYGROUND

在线IDE：https://beta.solpg.io/

## 案例1：一个简单的加法

### 合约编写 - ADD

```rust
use anchor_lang::prelude::*;

// 声明程序的链上地址(program ID)
declare_id!("2UF43ZNJtWzMqo4yJu69dmrSrmiky9wrAhWjPxmB5GCC");

// 处理指令的Solana程序模块
#[program]
mod Adder {
    use super::*;
    pub fn add(ctx: Context<Add>, d1: u64, d2: u64) -> Result<()> {
        msg!("Sum is: {}!", d1 + d2);
        Ok(())
    }
}

// 指令所需的账户列表，包装在Context结构中
#[derive(Accounts)]
    pub struct Add<> {
}
```

### 查看程序信息

```
# 查看某个Program的状态
solana program show <PROGRAM_ID> -u http://127.0.0.1:8899
# 查看账户的基本信息
solana account <PROGRAM_ID or address> -u http://127.0.0.1:8899
solana account <ACCOUNT> -u localhost
```

### 交互

交互后，在命令行可以看到交易指令及日志

```
# 监听程序的日志
solana logs <programID> -u http://127.0.0.1:8899
```

## 案例2：创建账户存数据

```rust
use anchor_lang::prelude::*;

declare_id!("GftF7uHmLLkFStK8t4dJzBHs21WJMpCf58wZGobRUqKk");

#[program]
mod hello_anchor {
    use super::*;
    pub fn initialize(ctx: Context<Initialize>, data: u64) -> Result<()> {
        ctx.accounts.new_account.data = data;
        msg!("Changed data to: {}!", data);
        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize<'info> {
    // init:创建这个账号 payer:支付手续费 space:账户空间
    #[account(init, payer = signer, space = 8 + 8)]
    pub new_account: Account<'info, NewAccount>,

    #[account(mut)]
    pub signer: Signer<'info>,

    pub system_program: Program<'info, System>,
}

// 组织数据的存储，决定如何序列化
#[account]
pub struct NewAccount {
    data: u64,
}
```

data传入10并运行程序，查看newAccount的账户信息，可以看到二进制数据中有对应的0a：

```
solana account <ACCOUNT> -u localhost
```

输出结果：

```
Public Key: Add4xbACcEPofbBroDNHaKfkzkhHavkZ97sGEV6m4Bbp
Balance: 0.00100224 SOL
Owner: GftF7uHmLLkFStK8t4dJzBHs21WJMpCf58wZGobRUqKk
Executable: false
Rent Epoch: 18446744073709551615
Length: 16 (0x10) bytes
0000:   b0 5f 04 76  5b b1 7d e8  0a 00 00 00  00 00 00 00   ._.v[.}.........
```

`b0 5f 04 76  5b b1 7d e8`是Discriminator，表示一个NewAccount类型数据。

## 案例3：为每个用户各自的存储账户(PDA)，保存用户喜欢的数字和颜色

```rust
use anchor_lang::prelude::*;

declare_id!("GiX3q7fjSvsrtDmztjEUumfSRoSBKRRFYQixLkcLnEKa");

pub const ANCHOR_DISCRIMINATOR_SIZE: usize = 8;

#[program]
pub mod favorites {
    use super::*;
    pub fn set_favorites(
        context: Context<SetFavorites>, number: u64, color: String,
    ) -> Result<()> {
        msg!("Greetings from {}", context.program_id);
        let user_public_key = context.accounts.user.key();
        msg!("User {user_public_key}'s favorite number is {number}, favorite color is: {color}",);

        context.accounts.favorites.set_inner(Favorites {
            number,
            color,
        });
        Ok(())
    }
}

#[derive(Accounts)]
pub struct SetFavorites<'info> {
    #[account(mut)]
    pub user: Signer<'info>,

    #[account(
        init_if_needed,
        payer = user,
        space = ANCHOR_DISCRIMINATOR_SIZE + Favorites::INIT_SPACE,
        seeds=[b"favorites", user.key().as_ref()],
        bump
    )]
    pub favorites: Account<'info, Favorites>,

    pub system_program: Program<'info, System>,
}

// What we will put inside the Favorites PDA
#[account]
#[derive(InitSpace)]
pub struct Favorites {
    pub number: u64,
    
    #[max_len(50)]
    pub color: String,
}
```

运行参数有两个seed，第一个是favorites，第二个是当前账户的public key。

开发文档地址：https://learnblockchain.cn/docs/anchor
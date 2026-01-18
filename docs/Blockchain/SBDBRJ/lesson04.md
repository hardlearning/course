# PROJECT - Write and test a custom Solana program

## Project intro + theory on custom programs and accounts

Program = piece of code that runs on the Solana blockchain

Program are "stateless"! Meaning you can't store data on them.

How do you store data in Solana? Use accounts!

Account = "file" that is stored on the Solana blockchain

## Set up an Anchor project

```
anchor init mycalculatordapp
cd mycalculatordapp
anchor build
anchor test
```

## Write the "create" function for your calculator

- programs/mycalculatordapp/src/lib.rs

```rust
use anchor_lang::prelude::*;

declare_id!("52tkEomCL4RayC3bqgVZk43YM6ZetxbAQRi9TmCWpavu");

#[program]
pub mod mycalculatordapp {
    use super::*;

    // 1. Write the "create" function for your calculator
    pub fn create(ctx: Context<Create>, init_message: String) -> Result<()> {
        let calculator = &mut ctx.accounts.calculator;
        calculator.greeting = init_message;
        Ok(())
    }
}

// 2. Specify the context of your "create" function
#[derive(Accounts)]
pub struct Create<'info> {
    #[account(init, payer=user, space=264)]
    pub calculator: Account<'info, Calculator>,
    #[account(mut)]
    pub user: Signer<'info>,
    pub system_program: Program<'info, System>
}

// 3. Specify the calculator account structure
#[account]
pub struct Calculator {
    pub greeting: String,
    pub result: i64,
    pub remainder: i64
}
```

## Write Mocha tests for your "create" function

- tests/mycalculatordapp.ts

```typescript
const assert = require('assert');
const anchor = require('@project-serum/anchor');
const {SystemProgram} = anchor.web3;

describe("mycalculatordapp", () => {
  const provider = anchor.Provider.local();
  anchor.setProvider(provider);
  const calculator = anchor.web3.Keypair.generate();
  const program = anchor.workspace.Mycalculatordapp;

  it("Create a calculator", async () => {
    await program.rpc.create("Welcome to Solana", {
      accounts: {
        calculator: calculator.publicKey,
        user: provider.wallet.publicKey,
        systemProgram: SystemProgram.programId
      },
      signers: [calculator]
    });
    const account = await program.account.calculator.fetch(calculator.publicKey);
    assert.ok(account.greeting === "Welcome to Solana");
  });
});
```
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
    pub fn create(ctx: Context<Create>, init_message: String) -> ProgramResult {
        let calculator = &mut ctx.accounts.calculator;
        calculator.greeting = init_message;
        Ok(())
    }
    // Write the addition function for your calculator
    pub fn add(ctx: Context<Addition>, num1: i64, num2: i64) -> ProgramResult {
        let calculator = &mut ctx.accounts.calculator;
        calculator.result = num1 + num2;
        OK(())
    }

    // IMPLEMENT YOURSELF! Subtraction function
    pub fn subtract(ctx: Context<Subtraction>, num1: i64, num2: i64) -> ProgramResult {
        let calculator = &mut ctx.accounts.calculator;
        calculator.result = num1 + num2;
        OK(())
    }

    // IMPLEMENT YOURSELF! Multiplication function
    pub fn multiply(ctx: Context<Multiplication>, num1: i64, num2: i64) -> ProgramResult {
        let calculator = &mut ctx.accounts.calculator;
        calculator.result = num1 * num2;
        OK(())
    }

    // IMPLEMENT YOURSELF! Division function
    pub fn divide(ctx: Context<Division>, num1: i64, num2: i64) -> ProgramResult {
        let calculator = &mut ctx.accounts.calculator;
        calculator.result = num1 / num2;
        calculator.remainder = num1 % num2;
        OK(())
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

#[derive(Accounts)]
pub struct Addition<'info> {
    #[account(mut)]
    pub calculator: Account<'info, Calculator>
}

// IMPLEMENT YOURSELF! Subtraction context
#[derive(Accounts)]
pub struct Subtraction<'info> {
    #[account(mut)]
    pub calculator: Account<'info, Calculator>
}

// IMPLEMENT YOURSELF! Multiplication context
#[derive(Accounts)]
pub struct Multiplication<'info> {
    #[account(mut)]
    pub calculator: Account<'info, Calculator>
}

// IMPLEMENT YOURSELF! Division context
#[derive(Accounts)]
pub struct Division<'info> {
    #[account(mut)]
    pub calculator: Account<'info, Calculator>
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

  // Write Mocha tests for your addition function
  it("Adds two numbers", async()=>{
    await program.rpc.add(new anchor.BN(2), new anchor.BN(3), {
      accouts: {
        calculator: calculator.publicKey
      }
    });
    const account = await program.account.calculator.fetch(calculator.publicKey);
    assert.ok(account.result.eq(new anchor.BN(5)))
  });

  // IMPLEMENT YOURSELF! Subtraction test
  it("Subtracts two numbers", async function(){
    await program.rpc.subtract(new anchor.BN(32), new anchor.BN(33), {
      accouts: {
        calculator: calculator.publicKey
      }
    });
    const account = await program.account.calculator.fetch(calculator.publicKey);
    assert.ok(account.result.eq(new anchor.BN(-1)));
  });

  // IMPLEMENT YOURSELF! Multiplication test
  it("Multiplies two numbers", async function(){
    await program.rpc.multiply(new anchor.BN(2), new anchor.BN(3), {
      accouts: {
        calculator: calculator.publicKey
      }
    });
    const account = await program.account.calculator.fetch(calculator.publicKey);
    assert.ok(account.result.eq(new anchor.BN(6)));
  });

  // IMPLEMENT YOURSELF! Division test
  it("Divides two numbers", async function(){
    await program.rpc.divide(new anchor.BN(10), new anchor.BN(3), {
      accouts: {
        calculator: calculator.publicKey
      }
    });
    const account = await program.account.calculator.fetch(calculator.publicKey);
    assert.ok(account.result.eq(new anchor.BN(3)));
    assert.ok(account.remainder.eq(new anchor.BN(1)));
  });
});
```

```
cd mycalculatordapp
solana address
solana airdrop 2 --url devnet
anchor test
```
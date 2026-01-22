# DEFI PROJECT - Lauch GoFundMe as a Web3 dapp

## Project intro + theory on program-derived accounts

Person A gives approval to send money from wallet A

Program gives approval to send money from account

Program-devived accounts can receive and send money. They are the backbone of Solana DeFi.

## Set up an Anchor project

```
anchor init crowdfunding
cd crowdfunding
anchor build
```

## Create a crowdfunding campaign

- programs/crowdfunding/src/lib.rs

```rust
use anchor_lang::prelude::*;
use anchor_lang::solana_program::entrypoint::ProgramResult;

declare_id!("BPcHXB5mUK7pKWuYFdfSYcPwQsTirRZHXJa7RU7KyMdt");

#[program]
pub mod crowdfunding {
    use super::*;

    pub fn create(ctx: Context<Create>, name: String, description: String) -> ProgramResult {
        let campaign = &mut ctx.accounts.campaign;
        campaign.name = name;
        campaign.description = description;
        campaign.amout_donated = 0;
        campaign.admin = *ctx.accounts.user.key;
        Ok(())
    }

    // 3.Withdraw money from a crowdfunding campaign
    pub fn withdraw(ctx: Context<Withdraw>, amount: u64) -> ProgramResult {
        let campaign = &mut ctx.accounts.campaign;
        let user = &mut ctx.accounts.user;
        if campaign.admin != *user.key {
            return Err(ProgramError::IncorrectProgramId);
        }
        let rent_balance = Rent::get()?.minimum_balance(campaign.to_account_info().data_len());
        if **campaign.to_account_info().lamports.borrow() - rent_balance < amount {
            return Err(ProgramError::IncorrectProgramId);
        }
        **campaign.to_account_info().try_borrow_mut_lamports()? -= amount;
        **user.to_account_info().try_borrow_mut_lamports()? += amount;
        Ok(())
    }
}

// 1.Specify the context for the "create" function
#[derive(Accounts)]
pub struct Create<'info> {
    #[account(init, payer=user, space=9000, seeds=[b"CAMPAIGN_DEMO".as_ref(), user.key().as_ref()], bump)]
    pub campaign: Account<'info, Campaign>,
    #[account(mut)]
    pub user: Signer<'info>,
    pub system_program: Program<'info, System>
}

#[derive(Accounts)]
pub struct Withdraw<'info> {
    #[account(mut)]
    pub campaign: Account<'info, Campaign>,
    #[account(mut)]
    pub user: Signer<'info>
}

// 2.Define the structure of a crowdfunding campaign
#[account]
pub struct Campaign {
    pub admin: Pubkey,
    pub name: String,
    pub description: String,
    pub amout_donated: u64
}
```

```
anchor build
```
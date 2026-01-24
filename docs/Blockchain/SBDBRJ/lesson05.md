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
        campaign.amount_donated = 0;
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

    // 4.Donate money to a crowdfunding campaign
    pub fn donate(ctx: Context<Donate>, amount: u64) -> ProgramResult {
        let ix = anchor_lang::solana_program::system_instruction::transfer(
            &ctx.accounts.user.key(),
            &ctx.accounts.campaign.key(),
            amount
        );
        anchor_lang::solana_program::program::invoke(
            &ix,
            &[
                ctx.accounts.user.to_account_info(),
                ctx.accounts.campaign.to_account_info()
            ]
        );
        (&mut ctx.accounts.campaign).amount_donated+=amount;
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

#[derive(Accounts)]
pub struct Donate<'info> {
    #[account(mut)]
    pub campaign: Account<'info, Campaign>,
    #[account(mut)]
    pub user: Signer<'info>,
    pub system_program: Program<'info, System>
}

// 2.Define the structure of a crowdfunding campaign
#[account]
pub struct Campaign {
    pub admin: Pubkey,
    pub name: String,
    pub description: String,
    pub amount_donated: u64
}
```

```
anchor build
```

## Deploy your dapp to the devnet

```
solana-keygen new -o id.json
solana airdrop 2 CDY8feXrKamCECqWxoK2nsyF8fpRHHsbBPtbGGXov6hp --url devnet
anchor build
# 这个地址粘贴到Anchor.toml的crowdfunding和lib.rs的declare_id中
solana address -k ./target/deploy/crowdfunding-keypair.json
# 重新build
anchor build
anchor deploy
# 解决NetworkUnreachable
anchor deploy -- --use-rpc
```

- Anchor.toml

```
[programs.localnet]
crowdfunding = "BPcHXB5mUK7pKWuYFdfSYcPwQsTirRZHXJa7RU7KyMdt"

[provider]
cluster = "devnet"
wallet = "./id.json"
```

生成的program id可以在[Solana Explorer](https://explorer.solana.com/)中查询

## Set up a blank React project

```
npx create-react-app frontend
cd frontend
npm install --save @solana/web3.js
npm install --save @project-serum/anchor
npm run start
```

## Add a "Connect wallet" button to your web app

- frontend/src/App.js

```javascript
import './App.css';
import { useEffect, useState } from "react";

const App = () => {
  const [walletAddress, setWalletAddress] = useState(null);
  const checkIfWalletIsConnected = async () => {
    try {
      const { solana } = window;
      if (solana) {
        if (solana.isPhantom) {
          console.log("Phantom wallet found!");
          const response = await solana.connect({
            onlyIfTrusted: true
          });
          console.log(
            "Connected with public key:",
            response.publicKey.toString()
          );
          setWalletAddress(response.publicKey.toString());
        }
      } else {
        alert("Solana object not found! Get a Phantom wallet");
      }
    } catch (error) {
      console.error(error);
    }
  };

  const connectWallet = async () => {
    const { solana } = window;
    if (solana) {
      const response = await solana.connect();
      console.log(
        "Connected with public key:",
        response.publicKey.toString()
      );
      setWalletAddress(response.publicKey.toString());
    }
  };

  const renderNotConnectedContainer = () => (
    <button onClick={connectWallet}>Connect to Wallet</button>
  )

  useEffect(() => {
    const onLoad = async () => {
      await checkIfWalletIsConnected();
    }
    window.addEventListener("load", onLoad);
    return () => window.removeEventListener("locad", onLoad);
  }, []);

  return (
    <div className='App'>
      {!walletAddress && renderNotConnectedContainer()}
    </div>
  );
};

export default App;
```

## Create a campaign from the web app

1. 复制target/idl/crowdfunding.json的内容到front/src/idl.json中
2. 修改App.js

```javascript
import './App.css';
import { useEffect, useState } from "react";
import idl from "./idl.json";
import { Connection, PublicKey, clusterApiUrl } from "@solana/web3.js";
import { Program, AnchorProvider, web3, utils, BN } from "@project-serum/anchor";
import { Buffer } from "buffer";
window.Buffer = Buffer;

const programID = new PublicKey(idl.address);
const network = clusterApiUrl("devnet");
const opts = {
  // processed: wait for transaction to be confirmed by the node where connected to.
  // finalized: if you want to be super sure the transaction.
  preflightCommitment: "processed"
};
const { SystemProgram } = web3;

const App = () => {
  const [walletAddress, setWalletAddress] = useState(null);
  const getProvider = () => {
    const connection = new Connection(network, opts.preflightCommitment);
    const provider = new AnchorProvider(
      connection,
      window.solana,
      opts.preflightCommitment
    );
    return provider;
  };
  const checkIfWalletIsConnected = async () => {
    try {
      const { solana } = window;
      if (solana) {
        if (solana.isPhantom) {
          console.log("Phantom wallet found!");
          const response = await solana.connect({
            onlyIfTrusted: true
          });
          console.log(
            "Connected with public key:",
            response.publicKey.toString()
          );
          setWalletAddress(response.publicKey.toString());
        }
      } else {
        alert("Solana object not found! Get a Phantom wallet");
      }
    } catch (error) {
      console.error(error);
    }
  };

  const connectWallet = async () => {
    const { solana } = window;
    if (solana) {
      const response = await solana.connect();
      console.log(
        "Connected with public key:",
        response.publicKey.toString()
      );
      setWalletAddress(response.publicKey.toString());
    }
  };

  const createCampaign = async () => {
    try {
      const provider = getProvider();
      const program = new Program(idl, programID, provider);
      const [campaign] = await PublicKey.findProgramAddress(
        [
          utils.bytes.utf8.encode("CAMPAIGN_DEMO"),
          provider.wallet.publicKey.toBuffer(),
        ],
        program.programId
      );
      await program.rpc.create("campaign name", "campaign description", {
        accounts: {
          campaign,
          user: provider.wallet.publicKey,
          systemProgram: SystemProgram.programId,
        },
      });
      console.log(
        "Created a new campaign w/ address:",
        campaign.toString()
      );
    } catch (error) {
      console.error("Error creating campaign account:", error);
    }
  };

  const renderNotConnectedContainer = () => (
    <button onClick={connectWallet}>Connect to Wallet</button>
  );

  const renderConnectedContainer = () => (
    <button onClick={createCampaign}>Create a campaign</button>
  );

  useEffect(() => {
    const onLoad = async () => {
      await checkIfWalletIsConnected();
    }
    window.addEventListener("load", onLoad);
    return () => window.removeEventListener("locad", onLoad);
  }, []);

  return (
    <div className='App'>
      {!walletAddress && renderNotConnectedContainer()}
      {walletAddress && renderConnectedContainer()}
    </div>
  );
};

export default App;
```

3. 解决报错：Module not found: Error: Can't resolve 'assert'

```
npm install --save assert
```

## Create a campaign from the web app

```javascript
import './App.css';
import { useEffect, useState } from "react";
import idl from "./idl.json";
import { Connection, PublicKey, clusterApiUrl } from "@solana/web3.js";
import { Program, AnchorProvider, web3, utils, BN } from "@project-serum/anchor";
import { Buffer } from "buffer";
import { program } from '@project-serum/anchor/dist/cjs/native/system';
window.Buffer = Buffer;

const programID = new PublicKey(idl.address);
const network = clusterApiUrl("devnet");
const opts = {
  // processed: wait for transaction to be confirmed by the node where connected to.
  // finalized: if you want to be super sure the transaction.
  preflightCommitment: "processed"
};
const { SystemProgram } = web3;

const App = () => {
  const [walletAddress, setWalletAddress] = useState(null);
  const [campaigns, setCampaigns] = useState([]);
  const getProvider = () => {
    const connection = new Connection(network, opts.preflightCommitment);
    const provider = new AnchorProvider(
      connection,
      window.solana,
      opts.preflightCommitment
    );
    return provider;
  };
  const checkIfWalletIsConnected = async () => {
    try {
      const { solana } = window;
      if (solana) {
        if (solana.isPhantom) {
          console.log("Phantom wallet found!");
          const response = await solana.connect({
            onlyIfTrusted: true
          });
          console.log(
            "Connected with public key:",
            response.publicKey.toString()
          );
          setWalletAddress(response.publicKey.toString());
        }
      } else {
        alert("Solana object not found! Get a Phantom wallet");
      }
    } catch (error) {
      console.error(error);
    }
  };

  const connectWallet = async () => {
    const { solana } = window;
    if (solana) {
      const response = await solana.connect();
      console.log(
        "Connected with public key:",
        response.publicKey.toString()
      );
      setWalletAddress(response.publicKey.toString());
    }
  };

  const getCampaigns = async() => {
    const connection = new Connection(network, opts.preflightCommitment);
    const provider = getProvider();
    Promise.all(
      (await connection.getProgramAccounts(programID)).map(
        async (campaign) => ({
          ...(await program.account.campaign.fetch(campaign.pubkey)),
          pubkey: campaign.pubkey
        })
      )
    ).then((campaigns) => setCampaigns(campaigns));
  };

  const createCampaign = async () => {
    try {
      const provider = getProvider();
      const program = new Program(idl, programID, provider);
      const [campaign] = await PublicKey.findProgramAddress(
        [
          utils.bytes.utf8.encode("CAMPAIGN_DEMO"),
          provider.wallet.publicKey.toBuffer(),
        ],
        program.programId
      );
      await program.rpc.create("campaign name", "campaign description", {
        accounts: {
          campaign,
          user: provider.wallet.publicKey,
          systemProgram: SystemProgram.programId,
        },
      });
      console.log(
        "Created a new campaign w/ address:",
        campaign.toString()
      );
    } catch (error) {
      console.error("Error creating campaign account:", error);
    }
  };

  const donate = async (publicKey) => {
    try {
      const provider = getProvider();
      const program = new Program(idl, programID, provider);
      
      await program.rpc.donate(new BN(0.2 * web3.LAMPORTS_PER_SOL), {
        accounts: {
          campaign: publicKey,
          user: provider.wallet.publicKey,
          systemProgram: SystemProgram.programId,
        },
      });
      console.log("Donated some money to:", publicKey.toString());
      getCampaigns();
    } catch (error) {
      console.error("Error donating:", error);
    }
  };

  const withdraw = async (publicKey) => {
    try {
      const provider = getProvider();
      const program = new Program(idl, programID, provider);
      await program.rpc.withdraw(new BN(0.2 * web3.LAMPORTS_PER_SOL), {
        accounts: {
          campaign: publicKey,
          user: provider.wallet.publicKey,
        },
      });
      console.log("Withdrew some money from:", publicKey.toString());
    } catch (error) {
      console.error("Error withdrawing:", error);
    }
  };

  const renderNotConnectedContainer = () => (
    <button onClick={connectWallet}>Connect to Wallet</button>
  );

  const renderConnectedContainer = () => (
    <>
      <button onClick={createCampaign}>Create a campaign</button>
      <button onClick={getCampaigns}>Get a list of campaigns</button>
      <br />
      {campaigns.map((campaign) => {
        <>
          <p>Campaign ID: {campaign.pubkey.toString()}</p>
          <p>
            Balance: {" "}
            {(
              campaign.amountDonated / web3.LAMPORTS_PER_SOL
            ).toString()}
          </p>
          <p>{campaign.name}</p>
          <p>{campaign.description}</p>
          <button onClick={()=>donate(campaign.pubkey)}>
            Click to donate
          </button>
          <button onClick={()=>withdraw(campaign.pubkey)}>
            Click to withdraw
          </button>
          <br />
        </>
      })}
    </>
  );

  useEffect(() => {
    const onLoad = async () => {
      await checkIfWalletIsConnected();
    }
    window.addEventListener("load", onLoad);
    return () => window.removeEventListener("locad", onLoad);
  }, []);

  return (
    <div className='App'>
      {!walletAddress && renderNotConnectedContainer()}
      {walletAddress && renderConnectedContainer()}
    </div>
  );
};

export default App;
```

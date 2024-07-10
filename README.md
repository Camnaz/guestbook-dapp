# Guestbook dApp Tutorial / Boilerplate

## Overview

This guide will walk you through creating a decentralized guestbook application using Hardhat for development, Solidity for smart contracts, and React for the frontend. This setup will be tailored for the Core network.

## Project Setup

### 1. Initialize the Project
Create a new directory for your project and navigate into it:

```sh
mkdir guestbook-dapp
cd guestbook-dapp
npm init --yes
```
<img width="1512" alt="gdscreenshot1" src="https://github.com/Camnaz/guestbook-dapp/assets/32852637/10da1acb-f824-4014-a058-fa4e85d6515f">


## 2. Install Dependencies
Install Hardhat and other necessary dependencies:

```
npm install --save-dev hardhat
npm install --save-dev chai @nomiclabs/hardhat-waffle
npm install react react-dom
```

<img width="1512" alt="gdscreenshot2" src="https://github.com/Camnaz/guestbook-dapp/assets/32852637/36812823-d36d-4c65-9ff2-700d5010edad">

3. Set Up Hardhat
Initialize a new Hardhat project:

```
npx hardhat
```

<img width="1512" alt="gdscreenshot3" src="https://github.com/Camnaz/guestbook-dapp/assets/32852637/9e95a66e-16eb-475f-8036-ba43385f6272">

Select "Create a Javascript Project" and "no" for installing the project's sample dependencies.

We will now open up our newly created hardhat project in a code editor, for this guide, we will be using Visual Studio Code.
navigate to the root of your directory in the terminal and type `code .` and hit enter

## 4. Create secret.json File
Create a secret.json file in the root directory of your project to store your private key securely. Replace YOUR_PRIVATE_KEY with your actual private key.

{
  "PrivateKey": "YOUR_PRIVATE_KEY"
}

<img width="1512" alt="gdscreenshot5" src="https://github.com/Camnaz/guestbook-dapp/assets/32852637/3b6b1921-e5c0-471c-ae7e-d4cad79f7fa0">


> [Refer to this guide](https://support.metamask.io/managing-my-wallet/secret-recovery-phrase-and-private-keys/how-to-export-an-accounts-private-key/) for instructions on how to export your account's private key via Metamask.

## 5. Update hardhat.config.js
Replace the contents of hardhat.config.js with the following configuration:

Ensure that the network settings are configured correctly for [Core Testnet](https://docs.coredao.org/docs/Dev-Guide/contract-verify#hardhat-verification)

```
require('@nomiclabs/hardhat-ethers');
require("@nomiclabs/hardhat-waffle");

const { PrivateKey } = require('./secret.json');

module.exports = {
  defaultNetwork: 'testnet',
  networks: {
    hardhat: {},
    testnet: {
      url: 'https://rpc.test.btcs.network',
      accounts: [PrivateKey],
      chainId: 1115,
    }
  },
  solidity: {
    compilers: [
      {
        version: '0.8.9',
        settings: {
          evmVersion: 'paris',
          optimizer: {
            enabled: true,
            runs: 200,
          },
        },
      },
    ],
  },
  paths: {
    sources: './contracts',
    cache: './cache',
    artifacts: './artifacts',
  },
  mocha: {
    timeout: 20000,
  },
};
```
<img width="1512" alt="gdscreenshot5 1" src="https://github.com/Camnaz/guestbook-dapp/assets/32852637/d78ae0d4-c9dc-4ba0-9e96-99c38c0a7c89">


## Writing the Smart Contract
1. Create the Guestbook Smart Contract

Delete the `lock.sol` file in `Contracts`
Create a file `contracts/Guestbook.sol` with the following content:

```
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.0 <0.9.0;

/// @title Guestbook
contract Guestbook {
    struct Entry {
        string name;
        string message;
    }

    Entry[] public entries;

    function signGuestbook(string memory _name, string memory _message) public {
        entries.push(Entry(_name, _message));
    }

    function getEntries() public view returns (Entry[] memory) {
        return entries;
    }
}
```
<img width="1512" alt="gdscreenshot6" src="https://github.com/Camnaz/guestbook-dapp/assets/32852637/61243ee5-8e24-4dd2-97fb-7e5bb301f220">


## Deploying the Smart Contract
1. Create Deployment Script
Create a file scripts/deploy.js with the following content:

```
const hre = require("hardhat");

async function main() {
  await hre.run('compile'); // Ensure the contracts are compiled

  const Guestbook = await hre.ethers.getContractFactory("Guestbook");
  const guestbook = await Guestbook.deploy();

  await guestbook.deployed();
  console.log("Guestbook contract deployed to:", guestbook.address);
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

<img width="1512" alt="gdscreenshot7" src="https://github.com/Camnaz/guestbook-dapp/assets/32852637/7520ccdc-c78d-4e14-8fc8-5c2d6d619573">


2. Deploy the Contract
Deploy the contract to the Core network:

```
npx hardhat run scripts/deploy.js --network testnet
```

<img width="1512" alt="gdscreenshot8" src="https://github.com/Camnaz/guestbook-dapp/assets/32852637/67c484f2-fd49-4bca-8688-f35be8361839">


## Setting Up the React Frontend
1. Create the React App
Create the basic structure for a React frontend:

```
npx create-react-app frontend
cd frontend
```

<img width="1512" alt="gdscreenshot9" src="https://github.com/Camnaz/guestbook-dapp/assets/32852637/31cfed96-9abf-4d2e-8bda-1fa2a8f1efa3">


2. Install Ethers.js
Install the Ethers.js library:

```
npm install ethers
```

<img width="1512" alt="gdscreenshot10" src="https://github.com/Camnaz/guestbook-dapp/assets/32852637/02027836-e117-4b1c-8324-e81194277b83">


3. Add GuestbookAbi.json
Copy the Guestbook.json file from artifacts/contracts/Guestbook.sol/ to the frontend/src directory and rename it to GuestbookAbi.json.

<img width="1512" alt="gdscreenshot11" src="https://github.com/Camnaz/guestbook-dapp/assets/32852637/1bf2a0c9-647c-49b9-90b5-011c354e2484">

<img width="1512" alt="gdscreenshot12" src="https://github.com/Camnaz/guestbook-dapp/assets/32852637/b6f4e0e2-e31b-4495-9738-f94586ee14e2">


5. Update frontend/src/App.js
Create or update the src/App.js file with the following content:

```
import React, { useState, useEffect } from 'react';
import { ethers } from 'ethers';
import GuestbookAbi from './GuestbookAbi.json';

const contractAddress = 'YOUR_CONTRACT_ADDRESS'; // Replace with your deployed contract address

function App() {
  const [provider, setProvider] = useState(null);
  const [contract, setContract] = useState(null);
  const [entries, setEntries] = useState([]);
  const [name, setName] = useState("");
  const [message, setMessage] = useState("");

  useEffect(() => {
    const init = async () => {
      if (window.ethereum) {
        try {
          console.log("Connecting to MetaMask...");
          const tempProvider = new ethers.providers.Web3Provider(window.ethereum);
          await tempProvider.send("eth_requestAccounts", []); // Request account access if needed
          setProvider(tempProvider);

          const signer = tempProvider.getSigner();
          console.log("Signer obtained:", signer);
          const tempContract = new ethers.Contract(contractAddress, GuestbookAbi, signer);
          setContract(tempContract);
          console.log("Contract initialized:", tempContract);

          const entries = await tempContract.getEntries();
          setEntries(entries);
          console.log("Entries loaded:", entries);
        } catch (error) {
          console.error("Error connecting to contract:", error);
        }
      } else {
        console.error("Please install MetaMask!");
      }
    };
    init();
  }, []);

  const signGuestbook = async () => {
    if (!contract) {
      console.error("Contract is not initialized!");
      return;
    }

    try {
      console.log("Signing guestbook with:", name, message);
      const tx = await contract.signGuestbook(name, message);
      console.log("Transaction sent:", tx);
      await tx.wait(); // Wait for the transaction to be mined
      console.log("Transaction mined:", tx);

      const updatedEntries = await contract.getEntries();
      setEntries(updatedEntries);
      console.log("Updated entries:", updatedEntries);
    } catch (error) {
      console.error("Error signing guestbook:", error);
    }
  };

  return (
    <div>
      <h1>Guestbook DApp</h1>
      <input 
        type="text" 
        placeholder="Your Name" 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
      />
      <input 
        type="text" 
        placeholder="Your Message" 
        value={message} 
        onChange={(e) => setMessage(e.target.value)} 
      />
      <button onClick={signGuestbook}>Sign Guestbook</button>
      <ul>
        {entries.map((entry, index) => (
          <li key={index}>
            <strong>{entry.name}:</strong> {entry.message}
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```
<img width="1512" alt="gdscreenshot13" src="https://github.com/Camnaz/guestbook-dapp/assets/32852637/e656f122-9cf5-4635-b28b-30d4b09cdb0b">

Start the React app:

>Ensure you are in /frontend within your repository (`cd frontend`)

```
npm start
```
You will be redirected to the development server

<img width="1512" alt="gdscreenshot14" src="https://github.com/Camnaz/guestbook-dapp/assets/32852637/c74e0b38-4c86-4af9-bfa2-f3aa9b9b52b3">

## Conclusion
By following these steps, you should have a basic decentralized guestbook application deployed on the Core network with a React frontend to interact with it. Ensure you replace placeholders like YOUR_CONTRACT_ADDRESS with actual values from your deployment.




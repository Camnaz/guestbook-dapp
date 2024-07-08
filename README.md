# Guestbook dApp

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

## 2. Install Dependencies
Install Hardhat and other necessary dependencies:

```
npm install --save-dev hardhat
npm install --save-dev chai @nomiclabs/hardhat-waffle
npm install react react-dom
```
3. Set Up Hardhat
Initialize a new Hardhat project:

```
npx hardhat
```

Select "Create an empty hardhat.config.js" and "no" for installing the project's sample dependencies.

## 4. Create secret.json File
Create a secret.json file in the root directory of your project to store your private key securely. Replace YOUR_PRIVATE_KEY with your actual private key.

{
  "PrivateKey": "YOUR_PRIVATE_KEY"
}

## 5. Update hardhat.config.js
Replace the contents of hardhat.config.js with the following configuration:

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
        version: '0.8.24',
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

## Writing the Smart Contract
1. Create the Guestbook Smart Contract
Create a file contracts/Guestbook.sol with the following content:

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

2. Deploy the Contract
Deploy the contract to the Core network:

```
npx hardhat run scripts/deploy.js --network testnet
```

## Setting Up the React Frontend
1. Create the React App
Create the basic structure for a React frontend:

```
npx create-react-app frontend
cd frontend
```

2. Install Ethers.js
Install the Ethers.js library:

```
npm install ethers
```

3. Add GuestbookAbi.json
Copy the Guestbook.json file from artifacts/contracts/Guestbook.sol/ to the frontend/src directory and rename it to GuestbookAbi.json.

4. Update src/App.js
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

Running the React App
Start the React app:

```
npm start
```

## Conclusion
By following these steps, you should have a basic decentralized guestbook application deployed on the Core network with a React frontend to interact with it. Ensure you replace placeholders like YOUR_CONTRACT_ADDRESS with actual values from your deployment.

```
git init
```

2. Add All Files:

```
git add .
```

3. Commit Changes

git commit -m "Initial commit"

4. Create a New Repository on GitHub:
Go to GitHub and create a new repository named guestbook-dapp.








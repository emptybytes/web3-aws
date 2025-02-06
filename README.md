# Building a Scalable Decentralized Application (DApp) with AWS Integration

This project demonstrates how to integrate AWS cloud services with a decentralized application (DApp) that uses an ERC20 token smart contract. The DApp allows users to connect their MetaMask wallet, interact with the blockchain, and fetch token balances using AWS Lambda and API Gateway.

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture Diagram](#architecture-diagram)
3. [Features](#features)
4. [Prerequisites](#prerequisites)
5. [Setup Instructions](#setup-instructions)
   - [Smart Contract Deployment](#smart-contract-deployment)
   - [AWS Lambda Function](#aws-lambda-function)
   - [API Gateway](#api-gateway)
   - [Frontend Setup](#frontend-setup)
6. [How It Works](#how-it-works)
7. [Tools and Technologies](#tools-and-technologies)
8. [Contributing](#contributing)
9. [License](#license)

---

## Overview

This project integrates AWS cloud services with a decentralized application (DApp) that uses an ERC20 token smart contract deployed on the Ethereum Sepolia testnet. The frontend is built using React.js and connects to MetaMask for wallet interaction. AWS Lambda and API Gateway handle backend logic and expose an HTTP endpoint for the frontend to interact with the blockchain.

---

## Architecture Diagram

```plaintext
+-------------------+       +-------------------+       +-------------------+
|                   |       |                   |       |                   |
|   User Browser    |       |   MetaMask Wallet |       |   Blockchain      |
|  (React Frontend) |<----->|   (Sepolia Testnet)|<----->|   (ERC20 Smart     |
|                   |       |                   |       |   Contract)       |
|  - Connect Wallet |       | - Sign Transactions|      |                   |
|  - Fetch Balance  |       |                   |       |                   |
+-------------------+       +-------------------+       +-------------------+
          |                           ^                           ^
          |                           |                           |
          v                           |                           |
+-------------------+                 |                           |
|                   |                 |                           |
|   API Gateway     |                 |                           |
|   (AWS Service)   |                 |                           |
|                   |                 |                           |
+-------------------+                 |                           |
          |                           |                           |
          v                           |                           |
+-------------------+                 |                           |
|                   |                 |                           |
|   AWS Lambda      |                 |                           |
|   (Node.js)       |-----------------+                           |
|                   |                                             |
+-------------------+                                             |
          |                                                       |
          v                                                       |
+-------------------+                                             |
|                   |                                             |
|   Web3.js Library |---------------------------------------------+
|   (Blockchain     |
|   Interaction)    |
+-------------------+
```

---

## Features

- **MetaMask Integration**: Users can connect their MetaMask wallet to the DApp.
- **Token Balance Query**: Fetch the ERC20 token balance of a connected wallet address.
- **AWS Lambda Backend**: Serverless backend to handle blockchain interactions.
- **API Gateway**: Exposes the AWS Lambda function as an HTTP endpoint for the frontend.
- **Scalable and Secure**: Leverages AWS serverless architecture for scalability and security.

---

## Prerequisites

Before running the project, ensure you have the following:

1. **MetaMask Wallet**:
   - Installed in your browser.
   - Configured to use the Goerli testnet.
   - Funded with test ETH from a faucet (e.g., [https://goerlifaucet.com](https://cloud.google.com/application/web3/faucet/ethereum/sepolia)).

2. **AWS Account**:
   - Access to AWS Lambda, API Gateway, and IAM roles.

3. **Node.js and npm**:
   - Required for running the React frontend.

4. **Infura Project**:
   - Create an Infura account and obtain a project ID for connecting to the Ethereum network.

5. **Solidity Compiler**:
   - Install Hardhat or Remix IDE for deploying the smart contract.

---

## Setup Instructions

### Smart Contract Deployment

1. **Write the ERC20 Token Contract**:
   ```solidity
   // SPDX-License-Identifier: MIT
   pragma solidity ^0.8.0;

   import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

   contract MyToken is ERC20 {
       constructor(uint256 initialSupply) ERC20("MyToken", "MTK") {
           _mint(msg.sender, initialSupply);
       }
   }
   ```

2. **Deploy the Contract**:
   - Use Remix IDE or Hardhat to deploy the contract on the Goerli testnet.
   - Note the contract address and ABI after deployment.

---

### AWS Lambda Function

1. **Create the Lambda Function**:
   - Go to the AWS Management Console > Lambda > Create Function.
   - Choose Node.js runtime and configure permissions.

2. **Lambda Code**:
   ```javascript
   const Web3 = require('web3');
   const ABI = [/* Paste your ERC20 ABI here */];
   const CONTRACT_ADDRESS = 'YOUR_CONTRACT_ADDRESS';

   exports.handler = async (event) => {
       const web3 = new Web3('https://goerli.infura.io/v3/YOUR_INFURA_PROJECT_ID');
       const contract = new web3.eth.Contract(ABI, CONTRACT_ADDRESS);

       try {
           const userAddress = event.address;
           const balance = await contract.methods.balanceOf(userAddress).call();
           return {
               statusCode: 200,
               body: JSON.stringify({ balance }),
           };
       } catch (error) {
           console.error(error);
           return {
               statusCode: 500,
               body: JSON.stringify({ error: 'Failed to fetch balance' }),
           };
       }
   };
   ```

3. **Install Dependencies**:
   - Package `web3` with your Lambda deployment or use AWS Lambda Layers.

---

### API Gateway

1. **Create the API**:
   - Go to the AWS Management Console > API Gateway > Create API.
   - Choose HTTP API and integrate it with the Lambda function.

2. **Deploy the API**:
   - Deploy the API and note the endpoint URL.

---

### Frontend Setup

1. **Create React App**:
   ```bash
   npx create-react-app dapp-frontend
   cd dapp-frontend
   npm install web3 axios
   ```

2. **Frontend Code**:
   ```javascript
   import React, { useState } from 'react';
   import Web3 from 'web3';
   import axios from 'axios';

   const App = () => {
       const [account, setAccount] = useState('');
       const [balance, setBalance] = useState('');

       const connectWallet = async () => {
           if (window.ethereum) {
               const web3 = new Web3(window.ethereum);
               await window.ethereum.request({ method: 'eth_requestAccounts' });
               const accounts = await web3.eth.getAccounts();
               setAccount(accounts[0]);
           } else {
               alert('Please install MetaMask!');
           }
       };

       const fetchBalance = async () => {
           try {
               const response = await axios.post('https://your-api-gateway-url.amazonaws.com/default/BlockchainHandler', {
                   address: account,
               });
               setBalance(response.data.balance);
           } catch (error) {
               console.error(error);
               alert('Failed to fetch balance');
           }
       };

       return (
           <div>
               <h1>ERC20 DApp</h1>
               <button onClick={connectWallet}>Connect Wallet</button>
               {account && <p>Connected Account: {account}</p>}
               <button onClick={fetchBalance} disabled={!account}>Fetch Balance</button>
               {balance && <p>Token Balance: {balance}</p>}
           </div>
       );
   };

   export default App;
   ```

3. **Run the App**:
   ```bash
   npm start
   ```

---

## How It Works

1. The user connects their MetaMask wallet to the React app.
2. The frontend sends the user's Ethereum address to the AWS API Gateway.
3. The API Gateway forwards the request to the AWS Lambda function.
4. The Lambda function queries the ERC20 smart contract for the token balance.
5. The response is sent back to the frontend, which displays the balance.

---

## Tools and Technologies

| Component               | Technology/Tool              |
|-------------------------|------------------------------|
| Frontend                | React.js, Axios             |
| Wallet Integration      | MetaMask                    |
| Backend                 | AWS Lambda (Node.js)        |
| API Exposure            | AWS API Gateway             |
| Blockchain Interaction  | Web3.js, Infura             |
| Smart Contract          | Solidity, OpenZeppelin      |
| Blockchain Network      | Ethereum (Sepolia Testnet)   |

---

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository.
2. Create a new branch (`git checkout -b feature/YourFeatureName`).
3. Commit your changes (`git commit -m 'Add some feature'`).
4. Push to the branch (`git push origin feature/YourFeatureName`).
5. Open a pull request.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

Let me know if you need further assistance!

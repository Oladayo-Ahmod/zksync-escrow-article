![zksync-logo_upscaled](https://github.com/Oladayo-Ahmod/escrow-contract/assets/57647734/2d905a0b-4434-4763-979b-ad06a14435d1)

# A Step-by-Step Guide to Building a Decentralized Escrow System on zkSync.

This article will delve into the intricate step-by-step process of building a decentralized escrow system that empowers users with secure, 
transparent, and efficient transactions on zkSync, a cutting-edge layer 2 scaling solution for Ethereum.

## Table of Contents:
-  [Introduction](#introduction)
-  [Prerequisites](#prerequisites)
-  [Environment Setup](#environment-setup)
-  [Understanding the Code Structure](#snderstanding-the-code)
   - [Project structure](#project-structure)
-  [Writing the Escrow Contract](#writing-the-escrow-contract)
   - [Register Purchaser](#register-purchaser)
   - [Register Vendor](#register-vendor)
   - [Create Agreement](#create-agreement)
   - [Enter Agreement](#enter-agreement)
   - [Deposit Funds](#deposit-funds)
   - [Release Payment](#release-payment)
   - [Refund Payment](#refund-payment)
-  [Complete Code](#complete-Code)
-  [Compiling Smart Contract](#compiling-smart-contract)
-  [Writing Tests](#writing-tests)
-  [Deploying Smart Contract](#deploying-smart-contract)

## Introduction

zkSync is a layer 2 scaling solution for Ethereum. It is designed not only to increase the transaction throughput of the Ethereum network by reducing transaction costs and latency but also to fully preserve its foundational values – freedom, self-sovereignty, and decentralization. It employs zero-knowledge proofs to achieve scalability and privacy. One of the greatest features of zkSync is hyperscalability which ensures that as transactional demands escalate, the system seamlessly accommodates them without compromising its robust security measures or incurring additional costs.

## Prerequisites

* Basic understanding of Solidity.
* Visual studio code (VS code) or remix ide.
* Faucet: Follow this guide to obtain zkSync faucet.
* Node js is installed on your machine.

## Environment Setup

zkSync provides easy ways to get started with setting your environment by providing great plugins on Hardhat and Foundry. You can get started by using use one of the two. However, for this tutorial, we will be using hardhat.

Run the below command inside your terminal to create the project with the necessary dependencies.

```bash
npx zksync-cli create demo --template hardhat_solidity
```
You will be prompted to enter your private key for the project. Enter your private or skip to set it later.

![escrow2](https://github.com/Oladayo-Ahmod/escrow-contract/assets/57647734/ece399b0-9fc6-492d-a4f7-4e84ee040119)

Next, you will prompted to select package manager. You can select the most comfortable one for you. However, for this tutorial, we will select npm.

![escrow3](https://github.com/Oladayo-Ahmod/escrow-contract/assets/57647734/879bc94e-a552-4096-a078-8e783d6e9a6d)

After selecting npm or your desired package manager, all the required dependencies will be installed.

## Understanding the Code Structure

### Project structure

#### 📁 Escrow-contract
- **📁 contracts**
  - **📁 erc20**
    - 📄 ERC20Token.sol
  - **📁 nft**
    - 📄 NFTContract.sol
  - **📁 paymasters**
    - 📄 Paymaster.sol
  - 📄 Greeter.sol
- **📁 deploy**
  - **📁 erc20**
    - 📄 deployERC20.ts
  - **📁 nft**
    - 📄 deployNFT.ts
  - 📄 deploy.ts
  - 📄 interact.ts
  - 📄 utils.ts
- **📁 test**
  - **📁 erc20**
    - 📄 erc20Token.test.ts
  - **📁 nft**
    - 📄 nftContract.test.ts
  - 📄 greeter.test.ts


Your folder structure should be similar to the one above. These are all pre-generated by `zksync-cli`.

Next, go to the terminal and navigate into the project directory to delete unrequired files inside the `contract` , `test` and `deploy` folder by running the command below:

```
cd escrow-contract && rm -rf ./contracts/* && rm -rf ./deploy/erc20 && rm -rf ./deploy/nft && rm -rf ./test/*
```
Finally, run npm install to install all the dependencies. If you are using yarn , run yarn install .

## Writing the Escrow Contract

Now that we have installed our project dependencies. Let's begin writing our smart contract.
Inside your `contract` folder, create a new file named Escrow.sol. This file will contain all the Escrow's contract codes.

#### SPDX License Identifier
In solidity, the first thing to declare when writing a smart contract is to declare the `spdx-lisense-identifier` .

```solidity
// SPDX-License-Identifier: MIT
```

#### Solidity Version
Next, let's define the contract pragma solidity version. In our case, 0.8.17.

```solidity
pragma solidity 0.8.17;
```

#### Contract's Name
Let's go ahead and name the contract Escrow.

```solidity
contract Escrow {
}
```

#### Variables
Let's define the state variables needed for the contract functionalities.

```solidity
    address public purchaser;
    address public vendor;
    address public intermediary;
    uint256 public totalAmount;
    bool public isFunded;
    bool public isFinished;
    uint8 totalAgreements;
```

Let's explain the usefulness of all these variables to the escrow contract.

- `address public purchaser;`: This variable stores the purchaser's address, the party who initiates the transaction.

- `address public vendor;`: This variable stores the vendor's address, the party who provides the goods or services.

- `address public intermediary;`: This variable stores the intermediary's address, a trusted third party who oversees the transaction process. In our case, the deployer of the contract.

- `uint256 public totalAmount;`: This variable keeps track of the total amount of funds deposited in the escrow.

- `bool public isFunded;`: This variable indicates whether the escrow has been funded or not.

- `bool public isFinished;`: This variable indicates whether the escrow transaction has been completed or not.

- `uint8 totalAgreements;`: This variable keeps track of the total agreements in the contract.

#### Mapping
Next, let's create a mapping for the Agreement struct.

```solidity
  mapping(uint256 => Agreement) public agreements;
```
This mapping makes it easy to store and retrieve agreements based on their unique identifiers.

#### Struct
Now, let's define the agreement's struct with the necessary fields.

```solidity
 struct Agreement{
  string title;
  string description;
  uint256 amount;
  address purchaser;
  address vendor;
 }
```
The `Agreement` struct defines the structure of an agreement in the escrow contract.
It contains fields such as title, description, amount, purchaser, and vendor to store essential information about the agreement.

#### Modifiers
Modifiers are very important to enforce access control in solidity. Let's go ahead and create these modifiers.

```solidity
  modifier onlyPurchaser() {
        require(msg.sender == purchaser, "Only the purchaser can call this function");
        _;
    }
    modifier onlyVendor() {
        require(msg.sender == vendor, "Only the vendor can call this function");
        _;
    }
    modifier onlyIntermediary() {
        require(msg.sender == intermediary, "Only the intermediary can call this function");
        _;
    }
    modifier onlyPurchaserOrIntermediary() {
        require(msg.sender == purchaser || msg.sender == intermediary, "Only the purchaser or intermediary can call this function");
        _;
    }
    modifier onlyNotFinished() {
        require(!isFinished, "Escrow has already been completed");
        _;
    }
```

Let's explain the functions of these modifiers.

- `modifier onlyPurchaser()`: This modifier ensures that only the `purchaser` can call the function it modifies by checking if the sender's address matches the address of the purchaser.

- `modifier onlyVendor()`: This modifier ensures that only the `vendor` can call the function it modifies by checking if the sender's address matches the address of the vendor.

- `modifier onlyIntermediary()`: This modifier ensures that only the `intermediary` can call the function it modifies by checking if the sender's address matches the address of the intermediary.

- `modifier onlyPurchaserOrIntermediary()`: This modifier allows either the `purchaser` or the `intermediary` to call the function it modifies by checking if the sender's address matches the address of the `purchaser` or the `intermediary`.

- `modifier onlyNotFinished()`: This modifier ensures that the function it modifies can only be called if the escrow has not been completed yet. It checks if the variable `isFinished` is false, indicating that the escrow is not finished yet.

#### Events
Events are very important in smart contracts. We will be using these events to monitor the behaviors of our contract and make it easy to communicate with external applications and off-chain systems.

```solidity
    event DepositMade(address indexed depositor, uint256 amount);
    event PaymentReleased(address indexed recipient, uint256 amount);
    event EscrowClosed();
```
Below are the explanations of these events:

- `event DepositMade(address indexed depositor, uint256 amount);`: This event is emitted when a deposit is made into the escrow. It includes the address of the depositor (purchaser) and the amount deposited.

- `event PaymentReleased(address indexed recipient, uint256 amount);`: This event is emitted when funds are released from the escrow. It includes the address of the recipient (vendor) and the amount released.

- `event EscrowClosed();`: This event is emitted when the escrow is closed, indicating that the transaction process has been completed or terminated.

#### Constructor
```solidity
  constructor() {
        intermediary = msg.sender;
    }
```

By default, the constructor sets whoever deploys the contract as the intermediary who oversees the transaction processes.

### Register Purchaser
Now, let's create a function to enable a new user to register a purchaser.
```solidity
 function registerPurchaser() external onlyNotFinished {
        require(purchaser == address(0), "Purchaser already registered");
        require(msg.sender != vendor, "Address is already registered as a vendor");
        purchaser = msg.sender;
    }
```
The `registerPurchaser` function allows the caller to register as a purchaser in the escrow contract. It checks if there is no purchaser currently registered in the contract and if the caller is not already registered as a vendor.

### Register Vendor
Let's also create a function to enable a new user to join as a vendor.

```solidity
 function registerVendor() external onlyNotFinished {
        require(vendor == address(0), "Vendor already registered");
        require(msg.sender != purchaser, "Address is already registered as a purchaser");
        vendor = msg.sender;
    }
```
The `registerVendor` function allows the caller to register as a vendor in the escrow contract if no vendor is currently registered and if the caller is not already registered as the purchaser.

### Create Agreement
Next, let's define a function that allows `vendor` to create an agreement.

```solidity
 function createAgreement(string memory _title, string memory _description, uint256 _amount) external onlyVendor{
        totalAgreements++;
        uint8 agreementId = totalAgreements;
        agreements[agreementId] = Agreement(_title,_description,_amount,address(0),msg.sender);
    }
```
This function allows a vendor to create a new agreement in the escrow contract. It increments the total number of agreements, assigns a unique ID to the new agreement, initializes its details, and stores it in the agreements mapping.

### Enter Agreement
After creating functionality to create an agreement with a vendor, let's go ahead to define a function that allows purchaser enters agreement.

```solidity
  function enterAgreement(uint8 _agreementId) external onlyPurchaser{
        require(_agreementId <= totalAgreements && _agreementId > 0, "agreement not found");
        Agreement storage agreement = agreements[_agreementId]; 
        agreement.purchaser = msg.sender;
    }
```
This function allows a purchaser to enter an existing agreement by entering `_agreementId`. It verifies the validity of the agreement ID and then assigns the purchaser's address to the agreement.

### Deposit Funds
It is important to create a function that allows a purchaser to deposit into the escrow.

```solidity
 function depositFunds(uint8 _agreementId) external onlyPurchaser payable onlyNotFinished {
        require(!isFunded, "Funds have already been deposited");
        require(_agreementId <= totalAgreements && _agreementId > 0, "agreement not found");
        Agreement storage agreement = agreements[_agreementId];
        uint budgetedAmount = agreement.amount;
        require(msg.value >= budgetedAmount, "Invalid deposit amount");
        totalAmount += msg.value;
        isFunded = true;
        emit DepositMade(purchaser, totalAmount);
    }
```
This function allows a purchaser to deposit funds into the escrow for a specific agreement he enters. It verifies that funds have not already been deposited, checks the validity of the agreement ID, ensures the deposited amount meets the budgeted amount specified in the agreement, updates the total amount stored in the escrow, sets the `isFunded` flag to true, and emits an event about the deposit.

### Release Payment
Let's create a function that allows the intermediary release funds from the escrow to the vendor.

```solidity
function releasePayment(uint8 _agreementId) external onlyIntermediary onlyNotFinished payable  {
        require(isFunded, "Funds must be deposited before releasing");
        require(_agreementId <= totalAgreements && _agreementId > 0, "agreement not found");
        Agreement storage agreement = agreements[_agreementId];
        uint amount = agreement.amount;
        (bool success, ) = vendor.call{value :amount}("");
        require(success, "Transfer to vendor failed");
        isFunded = false;
        isFinished = true;
        emit PaymentReleased(vendor, totalAmount);
        emit EscrowClosed();
    }
```
The `releasePayment` function allows the intermediary i.e. the deployer of the contract to release funds from the escrow to the vendor for a specific agreement. It checks if funds have been deposited, validates the agreement ID, transfers the specified amount to the vendor, updates the state of the escrow, and emits events about the payment release and the closure of the escrow transaction.

### Refund Payment
Finally, let's create a function that handles refunds of payment.

```solidity
 function refundPayment(uint8 _agreementId) external onlyPurchaserOrIntermediary onlyNotFinished {
        require(isFunded, "Funds must be deposited before refunding");
        require(_agreementId <= totalAgreements && _agreementId > 0, "agreement not found");
        Agreement storage agreement = agreements[_agreementId];
        uint amount = agreement.amount;
        (bool success,) = purchaser.call{value : amount}("");
        require(success , "Refund failed to purchaser");
        isFunded = false;
        isFinished = true;
        emit PaymentReleased(purchaser, totalAmount);
        emit EscrowClosed();
    }
```
The `refundPayment` function allows either the purchaser or the intermediary to refund funds from the escrow to the purchaser for a specific agreement only if the escrow is not yet completed. It checks if funds have been deposited, validates the agreement ID, transfers the specified amount to the purchaser, updates the state of the escrow, and emits events about the payment release and the closure of the escrow transaction.

## Complete Code
```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.17;

contract SecureEscrow {
    address public purchaser;
    address public vendor;
    address public intermediary;
    uint256 public totalAmount;
    bool public isFunded;
    bool public isFinished;
    uint8 totalAgreements;

    event DepositMade(address indexed depositor, uint256 amount);
    event PaymentReleased(address indexed recipient, uint256 amount);
    event EscrowClosed();

    mapping(uint256 => Agreement) public agreements;

    modifier onlyPurchaser() {
        require(msg.sender == purchaser, "Only the purchaser can call this function");
        _;
    }

    modifier onlyVendor() {
        require(msg.sender == vendor, "Only the vendor can call this function");
        _;
    }

    modifier onlyIntermediary() {
        require(msg.sender == intermediary, "Only the intermediary can call this function");
        _;
    }

    modifier onlyPurchaserOrIntermediary() {
        require(msg.sender == purchaser || msg.sender == intermediary, "Only the purchaser or intermediary can call this function");
        _;
    }

    modifier onlyNotFinished() {
        require(!isFinished, "Escrow has already been completed");
        _;
    }

    struct Agreement{
        string title;
        string description;
        uint256 amount;
        address purchaser;
        address vendor;
    }

    constructor() {
        intermediary = msg.sender;
    }

    function createAgreement(string memory _title, string memory _description, uint256 _amount) external onlyVendor{
        totalAgreements++;
        uint8 agreementId = totalAgreements;
        agreements[agreementId] = Agreement(_title,_description,_amount,address(0),msg.sender);

    }

    function enterAgreement(uint8 _agreementId) external onlyPurchaser{
        require(_agreementId <= totalAgreements && _agreementId > 0, "agreement not found");
        Agreement storage agreement = agreements[_agreementId]; 
        agreement.purchaser = msg.sender;
    }

    function registerPurchaser() external onlyNotFinished {
        require(purchaser == address(0), "Purchaser already registered");
        require(msg.sender != vendor, "Address is already registered as a vendor");
        purchaser = msg.sender;
    }
 
    function registerVendor() external onlyNotFinished {
        require(vendor == address(0), "Vendor already registered");
        require(msg.sender != purchaser, "Address is already registered as a purchaser");
        vendor = msg.sender;
    }

    function depositFunds(uint8 _agreementId) external onlyPurchaser payable onlyNotFinished {
        require(!isFunded, "Funds have already been deposited");
        require(_agreementId <= totalAgreements && _agreementId > 0, "agreement not found");
        Agreement storage agreement = agreements[_agreementId];
        uint budgetedAmount = agreement.amount;
        require(msg.value >= budgetedAmount, "Invalid deposit amount");
        totalAmount += msg.value;
        isFunded = true;
        emit DepositMade(purchaser, totalAmount);
    }

    function releasePayment(uint8 _agreementId) external onlyIntermediary onlyNotFinished payable  {
        require(isFunded, "Funds must be deposited before releasing");
        require(_agreementId <= totalAgreements && _agreementId > 0, "agreement not found");
        Agreement storage agreement = agreements[_agreementId];
        uint amount = agreement.amount;
        (bool success, ) = vendor.call{value :amount}("");
        require(success, "Transfer to vendor failed");
        isFunded = false;
        isFinished = true;
        emit PaymentReleased(vendor, totalAmount);
        emit EscrowClosed();
    }

    function refundPayment(uint8 _agreementId) external onlyPurchaserOrIntermediary onlyNotFinished {
        require(isFunded, "Funds must be deposited before refunding");
        require(_agreementId <= totalAgreements && _agreementId > 0, "agreement not found");
        Agreement storage agreement = agreements[_agreementId];
        uint amount = agreement.amount;
        (bool success,) = purchaser.call{value : amount}("");
        require(success , "Refund failed to purchaser");
        isFunded = false;
        isFinished = true;
        emit PaymentReleased(purchaser, totalAmount);
        emit EscrowClosed();
    }

}
```
## Compiling Smart Contract
To begin compiling the smart contract, let's pay attention to the hardhat.config.ts located in the root directory of the project folder.

```typescript
import { HardhatUserConfig } from "hardhat/config";

import "@matterlabs/hardhat-zksync";

const config: HardhatUserConfig = {
  defaultNetwork: "zkSyncSepoliaTestnet",
  networks: {
    zkSyncSepoliaTestnet: {
      url: "https://sepolia.era.zksync.dev",
      ethNetwork: "sepolia",
      zksync: true,
      verifyURL: "https://explorer.sepolia.era.zksync.dev/contract_verification",
    },
    zkSyncMainnet: {
      url: "https://mainnet.era.zksync.io",
      ethNetwork: "mainnet",
      zksync: true,
      verifyURL: "https://zksync2-mainnet-explorer.zksync.io/contract_verification",
    },
    zkSyncGoerliTestnet: { // deprecated network
      url: "https://testnet.era.zksync.dev",
      ethNetwork: "goerli",
      zksync: true,
      verifyURL: "https://zksync2-testnet-explorer.zksync.dev/contract_verification",
    },
    dockerizedNode: {
      url: "http://localhost:3050",
      ethNetwork: "http://localhost:8545",
      zksync: true,
    },
    inMemoryNode: {
      url: "http://127.0.0.1:8011",
      ethNetwork: "localhost", // in-memory node doesn't support eth node; removing this line will cause an error
      zksync: true,
    },
    hardhat: {
      zksync: true,
    },
  },
  zksolc: {
    version: "latest",
    settings: {
      // find all available options in the official documentation
      // https://era.zksync.io/docs/tools/hardhat/hardhat-zksync-solc.html#configuration
    },
  },
  solidity: {
    version: "0.8.17",
  },
};

export default config;
```
By default, your hardhat.config.ts should be similar to the one above. This configuration file provides settings for the smart contract development environment.

Next, run the below command to compile the contract.

```
npm run compile
```
After running the command, your terminal should output a similar result to this.
![escrow3](https://github.com/Oladayo-Ahmod/escrow-contract/assets/57647734/8038c893-a33d-422a-968f-36972eb4da6c)

This shows that the escrow smart contract has been successfully compiled.

## Writing Tests
If you are on Windows, you need to install [WSL2](https://www.omgubuntu.co.uk/how-to-install-wsl2-on-windows-10) to run zkSync nodes. Otherwise, you can skip this section.

Go ahead and create a file named `escrow.test.ts` inside your test folder.

Next, paste the following code to carry out the testing.

```typescript
import { expect } from 'chai';
import { Contract, Wallet } from "zksync-ethers";
import { getWallet, deployContract, LOCAL_RICH_WALLETS } from '../deploy/utils';
import * as ethers from "ethers";

describe('Escrow Contract', ()=>{
    let escrowContract: Contract;
    let intermediary: Wallet;
    let vendor: Wallet;
    let purchaser: Wallet;

    before(async function () {
        intermediary = getWallet(LOCAL_RICH_WALLETS[0].privateKey);
        vendor = getWallet(LOCAL_RICH_WALLETS[1].privateKey);
        purchaser = getWallet(LOCAL_RICH_WALLETS[2].privateKey);
    
        escrowContract = await deployContract("Escrow", [], { wallet: intermediary, silent: true });
    });

    it('Should allow purchaser to register', async function () {
      await (escrowContract.connect(purchaser) as Contract).registerPurchaser();
      expect(await escrowContract.purchaser()).to.equal(purchaser.address);
    });

    it('Should allow vendor to register', async function () {
      await (escrowContract.connect(vendor) as Contract).registerVendor();
      expect(await escrowContract.vendor()).to.equal(vendor.address);
    });

    it('Should create an agreement', async function () {
        const title = 'Test Agreement';
        const description = 'This is a test agreement';
        const amount = ethers.parseEther('1');
    
        await (escrowContract.connect(vendor) as Contract).createAgreement(title, description, amount);
        const agreement = await escrowContract.agreements(1);
    
        expect(agreement.title).to.equal(title);
        expect(agreement.description).to.equal(description);
        expect(agreement.amount).to.equal(amount);
        expect(agreement.vendor).to.equal(await vendor.getAddress());
      });

      it('Should allow purchaser to enter an agreement', async function () {
        await (escrowContract.connect(purchaser)as Contract).enterAgreement(1);
    
        const agreement = await escrowContract.agreements(1);
        expect(agreement.purchaser).to.equal(await purchaser.getAddress());
      });

      
      it('Should allow purchaser to deposit funds', async function () {
        const amount = ethers.parseEther('1');
        await (escrowContract.connect(purchaser) as Contract).depositFunds(1, { value: amount })
        expect(await escrowContract.isFunded()).to.true
      });

      it('Should allow intermediary to release payment', async function () {
        await (escrowContract.connect(intermediary) as Contract).releasePayment(1)
        expect(await escrowContract.isFunded()).to.false
        expect(await escrowContract.isFinished()).to.true
      });

      it('Should not allow refund payment after escrow is finished', async function () {
        try {
            const release = await (escrowContract.connect(intermediary) as Contract).refundPayment(1)
            await release.wait()
            expect.fail('Expected payment to fail but it did not')
        } catch (error) {
            expect(error.message).to.include("Escrow has already been completed");
        }
        
      }) 

})
```

Next, run the below command in your terminal.

``` npm run test ```

You should see a similar output if all the tests passed.
![escrow4](https://github.com/Oladayo-Ahmod/escrow-contract/assets/57647734/e07775df-010b-493b-8787-8ee5333d8c4d)

## Deploying Smart Contract
A few things are required to do to deploy this smart contract to zkSync.

Firstly, go ahead and enter your private key in the `.env` file in the root directory.

``` WALLET_PRIVATE_KEY=YOUR-PRIVATE-KEY ```

Note: You should replace `YOUR-PRIVATE-KEY` with your actual private key and ensure you have already some [zksync faucets](https://docs.zksync.io/build/tooling/network-faucets.html).

Next, open `deploy.ts` in your deploy folder and replace it with the code below.

```typescript
import { deployContract } from "./utils";
export default async function () {
  const contractArtifactName = "Escrow";
  const constructorArguments = [];
  await deployContract(contractArtifactName, constructorArguments);
}
```

Finally, proceed to your terminal and run the deploy script by entering the following command.

``` npm run deploy```

Your output should be similar to the below output if it is successfully deployed. However, `estimated deployment cost`,`contract address`, and `verification ID` may be different.
![escrow5](https://github.com/Oladayo-Ahmod/escrow-contract/assets/57647734/32bbb365-db83-44a3-8cce-2275351d57ab)


Congratulation! You have successfully written, tested, and deployed a decentralized escrow system on zkSync.



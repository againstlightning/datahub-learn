# How to create a ERC1155 NFT in Celo Network with Hardhat.
In this tutorial we'll write a smart contract using the Solidity language and a contract from the Openzeppelin library for ERC1155 tokens. Using nodejs along with Hardhat we will compile the smart contract code and also test the contract before deploying it. After deploying the contract we will create a custom task within Hardhat to create a Celo account and deploy the contract to the Celo network using [DataHub](https://datahub.figment.io). Lastly, we are going to use a React application that will connect to the Celo wallet account and interact with the deployed smart contract. 

![Captura de Tela (39)](https://user-images.githubusercontent.com/52639395/114646091-01e71d00-9cb1-11eb-8acc-214c255d2d4c.png)

# About the Author
I'm Lucas Espinosa ,a C#/solidity developer.if you are interested in these topics follow me on Github.
- [GitHub](https://github.com/lucasespinosa28)
- [Twitter](https://twitter.com/lucas9espinosa)

# Prerequisites 
 - We must have NodeJS >= v12.0 installed, preferably the latest version or an LTS release.
 - Knowledge of JavaScript, Solidity and React is beneficial.

# 1 - Install Hardhat
Hardhat is a development environment that compiles, deploys, tests, and helps you to debug your Ethereum smart contracts. Hardhat can also be used to deploy to the Celo network because Celo also runs the EVM (Ethereum Virtual Machine). This means smart contracts which work on Ethereum will also work on Celo. For the purposes of this tutorial, we will assume that the reader understands how to initialize a new Node project with a package manager (`npm` or `yarn`). We will go over how to install and configure Hardhat now.

#### Installing Hardhat
```bash
npm install --save-dev hardhat
npm install --save-dev @nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-ethers ethers web3 @celo/contractkit
```

#### Creating a new Hardhat project
From within the project directory, run :
```bash
npx hardhat
```
Selecting 'Create a sample project' will allow Hardhat to start its installation process in the current directory. It will create subdirectories and put all of the necessary files in place to empower your project. 
```bash
888    888                      888 888               888
888    888                      888 888               888
888    888                      888 888               888
8888888888  8888b.  888d888 .d88888 88888b.   8888b.  888888
888    888     "88b 888P"  d88" 888 888 "88b     "88b 888
888    888 .d888888 888    888  888 888  888 .d888888 888
888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
888    888 "Y888888 888     "Y88888 888  888 "Y888888  "Y888
Welcome to Hardhat v2.1.2
? What do you want to do? ...
> Create a sample project
  Create an empty hardhat.config.js
  Quit
```


# 2 - Write a smart contract

### The OpenZeppelin ERC1155 library
ERC1155 is a novel token standard that aims to take the best from previous standards to create a fungibility-agnostic and gas-efficient token contract.
ERC1155 draws ideas from all of ERC20, ERC721, and ERC777. ERC1155s are commonly used in NFT collectible projects, although they are not typically viewed as 'fine art' it is not unreasonable to use this token standard for such purposes. We will examine the use case of a token meant specifically for use within our Tiny Village.

### Install the OpenZeppelin contracts and create your smart contract file
 ```bash
npm install @openzeppelin/contracts
touch  contracts/TinyVillage.sol
```
### Write your smart contract 
You'll need to add a license to your code  ```SPDX License Identifier: MIT```, the source code of the smart contract will be visible on the blockchain, it's a best practice to have a license on your code to avoid the problem if third parties use your code.  with ```pragma solidity ^0.8.0;``` you'll set a compilation version, the compile version of your code and libraries must be compatible, the ```import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol``` will get the ERC1155.sol file in ```node_modules``` and other solidity files that ERC1155 will need to compile. Create the ``` contract TinyVillage  is ERC1155``` to give the name of the contract and tell to use the ERC1155 library using ```ìs````

#### ⚠ If you are using Remix import the ERC1155 module using github
```solidity
import "https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC1155/ERC1155.sol";
```
 ```solidity
 //SPDX-License-Identifier: MIT
 pragma solidity ^0.8.0;
 import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
 contract TinyVillage  is ERC1155 {
}
```
### Write the NFT parth of smart contract
Inside the ```contract TinyVillage  is ERC1155``` create the Nfts's Ids  ```uint256 public constant VILLAGE```.
Create the list with ours NFTs
```solidity
 uint256 public constant VILLAGE = 0;
 uint256 public constant MINE = 1;
 uint256 public constant FARM = 2;
 uint256 public constant MILL = 3;
 uint256 public constant CASTLE = 4;
```
The name is useful to remember the Ids it is possible to pass just the number without saving in a variable.
```solidity
 uint256 public constant VILLAGE = 0;
 _mint(msg.sender,VILLAGE,1,"0x000");
//The code are the same 
 _mint(msg.sender,0,1,"0x000");
```

The constructor code is executed once when a contract is created and it's used to initialize contract state.
The parament in ```ERC1155``` is a URI that can include the string {id} which clients must replace with the actual token ID.
  ```solidity
 constructor() ERC1155("https://gateway.pinata.cloud/ipfs/QmTN32qBKYqnyvatqfnU8ra6cYUGNxpYziSddCatEmopLR/metadata/api/item/{id}.json") {
  }
```
For token ID 1 and uri ```https://gateway.pinata.cloud/ipfs/QmTN32qBKYqnyvatqfnU8ra6cYUGNxpYziSddCatEmopLR/metadata/api/item/{id}.json``` clients would replace ***{id}*** with ***1*** to retrieve JSON at https://gateway.pinata.cloud/ipfs/QmTN32qBKYqnyvatqfnU8ra6cYUGNxpYziSddCatEmopLR/metadata/api/item/1.json.
The JSON document for token ID 1 might look something like:
```json
{
  "name": "Mine",
  "description": "Mine inseda a tiny mount",
  "image": "https://gateway.pinata.cloud/ipfs/QmPoFKTD8U2Mg6kgMheyv9K4rtPRnKv78orRHjaYaTVqNm/images/mine.png"
}
```
### Write the function will mint the NFTs
Our function has 2 parts,the ***require*** that get balance and if the user of contract does not have yet the Nft the contract vai let mint a NFT with the ***_mint*** function,What the mint does creates amount tokens of token type id, and assigns them to account.
 ```solidity
//If you do not have any village the contract will let you buy one
function mintVillage() public{
 require(balanceOf(msg.sender,VILLAGE) == 0,"you already have a Village ");
 _mint(msg.sender,VILLAGE,1,"0x000");
}

//If you do not have any Mine and have Village the contract will let you buy the Mine
function mintMine() public{
 require(balanceOf(msg.sender,VILLAGE) > 0,"you need have a Village");
 require(balanceOf(msg.sender,MINE) == 0,"you already have a Mine");
 _mint(msg.sender,MINE,1,"0x000");
}

//If you do not have any Farm and have Village the contract will let you buy the Farm
function mintFarm() public{
 require(balanceOf(msg.sender,VILLAGE) > 0,"you need have a Village");
 require(balanceOf(msg.sender,FARM) == 0,"you already have a Farm");
 _mint(msg.sender,FARM,1,"0x000");
}
    
//If you do not have any Mill and have Village and Farm the contract will let you buy the Mill
function mintMill() public{
 require(balanceOf(msg.sender,VILLAGE) > 0,"you need have a Village");
 require(balanceOf(msg.sender,FARM) > 0,"you need have a Farm");
 require(balanceOf(msg.sender,MILL) == 0,"you already have a Mill");
 _mint(msg.sender,MILL,1,"0x000");
}

//If you do not have any Castle and have all others NFt the contract will let you buy the Mill
function mintCastle() public{
 require(balanceOf(msg.sender,MINE) > 0,"you need have a Mine");
 require(balanceOf(msg.sender,MILL) > 0,"you need have a Mill");
 require(balanceOf(msg.sender,CASTLE) == 0,"you already have a Castle");
 _mint(msg.sender,CASTLE,1,"0x000");
}
```
#### [TinyVillage.sol complete code ](https://github.com/lucasespinosa28/Celo-Tutorial/blob/main/demo/contracts/TinyVillage.sol)

### Use Harthat to compile
In order to transform our solidity code into a contract, we need to compile it, after compile will get bytecode and other information about the contract in ```artifacts\contracts\TinyVillage.sol\TinyVillage.json``` the result of compilation.

To compile with Hardhat,first,delete ```contracts/Greeter.sol``` and the ```test/sample-test.js```,Open the ```hardhat.config.js``` and set the same version that you are using in the **TinyVillage.sol** smart contract.
 ```javascript
module.exports = {
  solidity: "0.8.0",
};
```
To compile your smart contract just use the command: ```npx hardhat compile```.
 ```bash
npx hardhat compile

Solidity 0.8.0 is not fully supported yet. You can still use Hardhat, but some features, like stack traces, might not work correctly.
Learn more at https://hardhat.org/reference/solidity-support"
Compiling 9 files with 0.8.0
Compilation finished successfully
```
🔺If you don't have deleted the ```contracts/Greeter.sol``` it's will give an error because  that the contract doesn't have the same version.

# 3 - Test and Deploy with hardhat

⚠verify if the code was compilet right in ```artifacts\contracts\TinyVillage.sol\Tiny Village.json```, This JSON file have all information necessary to deploy a smart contract.

### Use Harthat to test you smart contract
The best practive is test the smart contract in your machine before deploy, Create the test file in ```test/tineyVillageTest.js```. The [hardhat documentation](https://hardhat.org/guides/truffle-testing.html) has more information about testing using other libraries. 

Import the module [Chai](https://www.chaijs.com/), it's are our test library,it was installed in the first part of this tutorial.
```javascript
const { expect } = require("chai");
```
The first test the most basic testing just deploying the contract and minting the village, if the Village is minted the contract will pass.
```javascript
describe("TinyVillage Test", function() {
    it("Should mint village", async function() {
        ```
        const accounts = await ethers.getSigners();

        const TinyVillage = await ethers.getContractFactory("TinyVillage");
        const tinyVillage = await TinyVillage.deploy();

        await tinyVillage.mintVillage();
        const balance = await tinyVillage.balanceOf(accounts[0].address,0)
        expect(1).to.equal(Number(balance.toString()));
    });
```
Testing the minting of a Castle would require us to mint every other type of NFT, but for the purposes of this tutorial we will cut it short and not cover testing all of the minting functions.
```javascript
    it("Should mint castle",async function () {
        const accounts = await ethers.getSigners();

        const TinyVillage = await ethers.getContractFactory("TinyVillage");
        const tinyVillage = await TinyVillage.deploy();

        await tinyVillage.mintVillage();
        await tinyVillage.mintMine();
        await tinyVillage.mintFarm();
        await tinyVillage.mintMill();
        await tinyVillage.mintCastle();
        const balance = await tinyVillage.balanceOf(accounts[0].address, 4)
        expect(1).to.equal(Number(balance.toString()));
    });
});
```
Now will run our tests using the hardhat,Depending of your machine the test time may be different , If all tests have passed your smart contract are ready to deploy.
```bash
 npx hardhat test
 
 TinyVillage Test
    √ Should mint village (5832ms)
    √ Should mint castle (2321ms)
  2 passing (8s)
```
#### [tineyVillageTest.js complete code ](https://github.com/lucasespinosa28/Celo-Tutorial/blob/main/demo/test/tineyVillageTest.js)

### Creat a Celo account with Hardhat
To deploy your contract on Celo Testnet will need to create an account and save the privateKey to get an address for sent Celo test coin,In main folder create a file ```celo_account.js```,We'll not use the default accounts, Because we'll need to save account to use later,

```javascript
const Web3 = require('web3')
const fs = require('fs')
const path = require('path')
const web3 = new Web3()
const privateKeyFile = path.join(__dirname, './.secret')

//Function getAccount will return the address of your account
const getAccount = () => {
    const secret = fs.readFileSync(privateKeyFile);
    const account = web3.eth.accounts.privateKeyToAccount(secret.toString())
    return account;
}

//Function setAccount will crate new account and save the privateKey in .secret file 
const setAccount = () => {
    const newAccount = web3.eth.accounts.create()
    fs.writeFileSync(privateKeyFile, newAccount.privateKey, (err) => {
        if (err) {
            console.log(err);
        }
    })
    console.log(`Address ${newAccount.address}`)
}
module.exports = {
    getAccount,
    setAccount
}
```
In ```hardhat.config.js``` will necessary import modules to read the .secret file and the code of ***celo_account.js***. 
```javascript
const fs = require('fs')
const path = require('path')
const privateKeyFile = path.join(__dirname, './.secret')
const Account = require('./celo_account');
```
Below of others task add a new task, If the .secret file does not exist the task will create a new account if the .secret file exists the task will use the private key to get an address. 
```javascript
task("celo-account", "Prints account address or create a new", async () => {
    fs.existsSync(privateKeyFile) ? console.log(`Address ${Account.getAccount().address}`) : Account.setAccount();
});
```
When run ```npx hardhat celo-account```, The new account are created and the privateKey will saved into .secret file,Is important save the address and go to this website  [Celo faucets ](https://celo.org/developers/faucets) https://celo.org/developers/faucet,get the coins will be needed  to pay the fee to deploy.
```bash
 npx hardhat celo-account
 
 Address <your new address>
```
###### [celo_account.js complete code ](https://github.com/lucasespinosa28/Celo-Tutorial/blob/main/demo/celo_account.js)
###### [hardhat.config.js complete code ](https://github.com/lucasespinosa28/Celo-Tutorial/blob/main/demo/hardhat.config.js)

### Deploy
If your code is already compiled and tested, It's time to deploy your smart contract,You will need to connect in Celo BlockChain, To do this you will need a server running the Celo network,To connect on Celo network  and another Blockchain network one of the easiest ways and using figment DataHub, in the [DataHub (figment.io)](https://datahub.figment.io/) and chose Celo, copy celo-alfajores--rpc.datahub.figment.io and save the api key ![Captura de Tela (43)](https://user-images.githubusercontent.com/52639395/114647928-46c08300-9cb4-11eb-90a4-5a8600c7696d.png),

### Create deploy task
Create ```celo_deploy.js``` and write the code bellow,```@celo/contractkit``` is a library to help developers and validators to interact with the celo-blockchain,```TinyVillage()``` is the function used to deploy TinyVillage contract,the ```/artifacts/contracts/TinyVillage.sol/TinyVillage.json```is the file that contains all the information about our compiled contract.
```javascript
const Web3 = require('web3')
const ContractKit = require('@celo/contractkit')

const web3 = new Web3('https://celo-alfajores--rpc.datahub.figment.io/apikey/<key>/')
const kit = ContractKit.newKitFromWeb3(web3)
const data = require('./artifacts/contracts/TinyVillage.sol/TinyVillage.json')
const Account = require('./celo_account');

async function TinyVillage() {
    const account = Account.getAccount()
    kit.connection.addAccount(account.privateKey) 
    let tx = await kit.connection.sendTransaction({
        from: account.address,
        data: data.bytecode
    })
     return tx.waitReceipt()
}
module.exports = {
    TinyVillage
}
```
The same you did in ```task("celo-account"}```,Create a task to deploy.
```javascript
const Deploy = require('./celo_deploy');
```
```javascript
task("celo-deploy", "Prints account address or create a new", async () => {
    const tx = await Deploy.TinyVillage();
    console.log(tx);
    console.log(`save the contract address ${tx.contractAddress}`)
});
```

Run the task ```npx hardhat celo-deploy``` to deploy, Then save the contract address to use in your react app to interact with the smart contract, After you run the deploy task if everything goes right the result should look like this, but with different address and hash.
```bash
npx hardhat celo-deploy

{
  blockHash: '0x7b653ecce5042d1424cd66a5ce36112c8ece51de50317851aa887ab7582d0cd3',
  blockNumber: 4729795,
  contractAddress: '0x47A424A0975924C3d177470C519C8DBA37e16Ec9',
  cumulativeGasUsed: 2856692,
  from: '0x69e5ba06aa6176854ad01cf4fe6fb88119c9e378',
  gasUsed: 2856692,
  logs: [],
  logsBloom: '0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
  status: true,
  to: null,
  transactionHash: '0x92bf51162e35ec0ebe984a67914f208720f5d73091978af346c413f2715c3601',
  transactionIndex: 0
}

save the contract address 0x47A424A0975924C3d177470C519C8DBA37e16Ec9
```

###### [celo_deploy.js complete code ](https://github.com/lucasespinosa28/Celo-Tutorial/blob/main/demo/celo_deploy.js)
###### [hardhat.config.js complete code ](https://github.com/lucasespinosa28/Celo-Tutorial/blob/main/demo/hardhat.config.js)


# 3 - Interacts with the contract deployed

Marketplace sites like  [opensea.io](https://opensea.io/)  do not display NFTs on Celo yet. To be able to view NFTs on Celo, we can build a React app. A sample project is available on [github](https://github.com/lucasespinosa28/Celo-Tutorial/tree/main/demo/start),the complate app is available on [github](https://github.com/lucasespinosa28/Celo-Tutorial/tree/main/demo/final)

### Show the Images 
Open the file ```src/Images.js``` and write the code in the place indicated by comment,the images are already saved in the app.
```javascript
//Access a the contract
const contract = new kit.web3.eth.Contract(data.abi, "0x47A424A0975924C3d177470C519C8DBA37e16Ec9")
//Array with address NFT's owner 
const ownerAddress = [address, address, address, address, address]
//Array with NFT's id      
const ownerIds = [0, 1, 2, 3, 4]
//Function that will return the NFT that you have
async function getBalance() {
 const balances = await contract.methods.balanceOfBatch(ownerAddress, ownerIds).call();
 setBalanceArray(balances);
 }
```
This code work accessing the smart contract using  ```kit.web3.eth.Contract...```,The code has 2 arrays one with user address and the other with list NFTs id , when call the function ***balance Of Batch*** in contract will return an array with each NFT that the user has,Depends on the balance the code will show a specific image for each array combination,example: if balanceOfBatch returns ***[1,0,0,0,0]*** the code will show this image
![village](https://user-images.githubusercontent.com/52639395/115546874-ee711e80-a27b-11eb-90e5-2febf733aabc.jpg)

### Create button to mint the NFT 
Open the file ```src/MintNFT.js```,this code work getting the balance like the ***Images.js***,Write the code below to show the buttons to mint the NFT
the code below work like the Images.js ,But it's create a button only if is possible to mint the NFT.
```javascript
 //Access a the contract
        const contract = new kit.web3.eth.Contract(data.abi, "0x47A424A0975924C3d177470C519C8DBA37e16Ec9")
        //Array with address NFT's owner 
        const ownerAddress = [address, address, address, address, address]
        //Array with NFT's id     
        const ownerIds = [0, 1, 2, 3, 4]
        //Function that will return the NFT that you have
        async function getBalance() {
            const data = await contract.methods.balanceOfBatch(ownerAddress, ownerIds).call();
            setBalanceArray(data);
        }
        // Run the code above 
        getBalance();
        //this function will coin NFT depending on the name you give 
        async function Mint(name) {
            console.log(balanceArray);
            contract.methods.[name]().send({ from: address })
                .on('transactionHash', function (hash) {
                    console.log(hash);
                })
                .on('receipt', function (receipt) {
                    console.log(receipt);
                })
                .on('error', function (error, receipt) {
                    console.log(error);
                    console.log(receipt);
                });
        }
```
To not create a function to mintVillage,mintFarm,mintMine and etc, The same function will mint an NFT using the name of the function it has in the smart contract ,
example:
```javascript
async function Mint("mintVillage") {
 contract.methods.["mintVillage"]().send(...
}
Mint("mintVillage")}
....
//if not using this method
contract.methods.mintVillage().send({ from: address })
contract.methods.mintCastle().send({ from: address })
....
```
If the code have any error, test the app.
```bash
npm start
```
![Captura de Tela (49)](https://user-images.githubusercontent.com/52639395/115549890-94725800-a27f-11eb-9993-c9ddc794eb12.png)
![Captura de Tela (50)](https://user-images.githubusercontent.com/52639395/115549907-99370c00-a27f-11eb-9b41-230732ca3155.png)
After you connect to a account go to [Celo faucets ](https://celo.org/developers/faucets) and send test coin to address to  buying the NFT,First, you need to buy a village and next, you will need to buy a farm and mine to pass to the next level.
![Captura de Tela (54)](https://user-images.githubusercontent.com/52639395/115550371-267a6080-a280-11eb-94ab-1840236fdbac.png)
![Captura de Tela (55)](https://user-images.githubusercontent.com/52639395/115550456-43169880-a280-11eb-94e3-e101d18722eb.png)

# Conclusion
We learning how writer [ERC1155](https://docs.openzeppelin.com/contracts/3.x/erc1155) contracts, use [hardhat](https://hardhat.org/) to create custom tasks and [celo contractkit](https://github.com/celo-tools/use-contractkit)to intereact with Celo contracts. 

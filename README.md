# rlw

Developing smart contracts
Welcome to the exciting world of smart contract development! This guide will let you get started writing Solidity contracts by going over the following:

Setting up a Solidity Project

Compiling Solidity Source Code

Adding More Contracts

Using OpenZeppelin Contracts

About Solidity
We won’t cover language concepts such as syntax or keywords in this guide. For that, you’ll want to check out the following curated content, which feature great learning resources for both newcomers and experienced developers:

For a general overview of how Ethereum and smart contracts work, the official website hosts a Learn about Ethereum section with lots of beginner-friendly content.

If you’re new to the language, the official Solidity documentation is a good resource to have handy. Take a look at their security recommendations, which nicely go over the differences between blockchains and traditional software platforms.

Consensys' best practices are quite extensive, and include both proven patterns to learn from and known pitfalls to avoid.

The Ethernaut web-based game will have you look for subtle vulnerabilities in smart contracts as you advance through levels of increasing difficulty.

With that out of the way, let’s get started!

Setting up a Project
The first step after creating a project is to install a development tool.

The most popular development framework for Ethereum is Hardhat, and we cover its most common use with ethers.js. The next most popular is Truffle which uses web3.js. Each has their strengths and it is useful to be comfortable using all of them.

In these guides we will show how to develop, test and deploy smart contracts using Truffle and Hardhat.

Instructions are available for both Truffle and Hardhat. Choose your preference using this toggle!

Toggle Hardhat or Truffle
To get started with Hardhat we will install it in our project directory.

$ npm install --save-dev hardhat
Once installed, we can run npx hardhat. This will create a Hardhat config file (hardhat.config.js) in our project directory.

$ npx hardhat
888    888                      888 888               888
888    888                      888 888               888
888    888                      888 888               888
8888888888  8888b.  888d888 .d88888 88888b.   8888b.  888888
888    888     "88b 888P"  d88" 888 888 "88b     "88b 888
888    888 .d888888 888    888  888 888  888 .d888888 888
888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
888    888 "Y888888 888     "Y88888 888  888 "Y888888  "Y888

Welcome to Hardhat v2.2.1

✔ What do you want to do? · Create an empty hardhat.config.js
Config file created
First contract
We store our Solidity source files (.sol) in a contracts directory. This is equivalent to the src directory you may be familiar with from other languages.

We can now write our first simple smart contract, called Box: it will let people store a value that can be later retrieved.

We will save this file as contracts/Box.sol. Each .sol file should have the code for a single contract, and be named after it.

// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Box {
    uint256 private _value;

    // Emitted when the stored value changes
    event ValueChanged(uint256 value);

    // Stores a new value in the contract
    function store(uint256 value) public {
        _value = value;
        emit ValueChanged(value);
    }

    // Reads the last stored value
    function retrieve() public view returns (uint256) {
        return _value;
    }
}
Compiling Solidity
The Ethereum Virtual Machine (EVM) cannot execute Solidity code directly: we first need to compile it into EVM bytecode.

Our Box.sol contract uses Solidity 0.8 so we need to first configure Hardhat to use an appropriate solc version.

We specify a Solidity 0.8 solc version in our hardhat.config.js.

// hardhat.config.js

/**
 * @type import('hardhat/config').HardhatUserConfig
 */
 module.exports = {
  solidity: "0.8.4",
};
Compiling can then be achieved by running a single compile command:

If you’re unfamiliar with the npx command, check out our Node project setup guide.
$ npx hardhat compile
Solidity 0.8.4 is not fully supported yet. You can still use Hardhat, but some features, like stack traces, might not work correctly.

Learn more at https://hardhat.org/reference/solidity-support"

Compiling 1 file with 0.8.4
Compilation finished successfully
The compile built-in task will automatically look for all contracts in the contracts directory, and compile them using the Solidity compiler using the configuration in hardhat.config.js.

You will notice an artifacts directory was created: it holds the compiled artifacts (bytecode and metadata), which are .json files. It’s a good idea to add this directory to your .gitignore.

Adding more contracts
As your project grows, you will begin to create more contracts that interact with each other: each one should be stored in its own .sol file.

To see how this looks, let’s add a simple access control system to our Box contract: we will store an administrator address in a contract called Auth, and only let Box be used by those accounts that Auth allows.

Because the compiler will pick up all files in the contracts directory and subdirectories, you are free to organize your code as you see fit. Here, we’ll store the Auth contract in an access-control subdirectory:

// contracts/access-control/Auth.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Auth {
    address private _administrator;

    constructor(address deployer) {
        // Make the deployer of the contract the administrator
        _administrator = deployer;
    }

    function isAdministrator(address user) public view returns (bool) {
        return user == _administrator;
    }
}
To use this contract from Box we use an import statement, referring to Auth by its relative path:

// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// Import Auth from the access-control subdirectory
import "./access-control/Auth.sol";

contract Box {
    uint256 private _value;
    Auth private _auth;

    event ValueChanged(uint256 value);

    constructor() {
        _auth = new Auth(msg.sender);
    }

    function store(uint256 value) public {
        // Require that the caller is registered as an administrator in Auth
        require(_auth.isAdministrator(msg.sender), "Unauthorized");

        _value = value;
        emit ValueChanged(value);
    }

    function retrieve() public view returns (uint256) {
        return _value;
    }
}
Separating concerns across multiple contracts is a great way to keep each one simple, and is generally a good practice.

However, this is not the only way to split your code into modules. You can also use inheritance for encapsulation and code reuse in Solidity, as we’ll see next.

Using OpenZeppelin Contracts
Reusable modules and libraries are the cornerstone of great software. OpenZeppelin Contracts contains lots of useful building blocks for smart contracts to build on. And you can rest easy when building on them: they’ve been the subject of multiple audits, with their security and correctness battle-tested.

About inheritance
Many of the contracts in the library are not standalone, that is, you’re not expected to deploy them as-is. Instead, you will use them as a starting point to build your own contracts by adding features to them. Solidity provides multiple inheritance as a mechanism to achieve this: take a look at the Solidity documentation for more details.

For example, the Ownable contract marks the deployer account as the contract’s owner, and provides a modifier called onlyOwner. When applied to a function, onlyOwner will cause all function calls that do not originate from the owner account to revert. Functions to transfer and renounce ownership are also available.

When used this way, inheritance becomes a powerful mechanism that allows for modularization, without forcing you to deploy and manage multiple contracts.

Importing OpenZeppelin Contracts
The latest published release of the OpenZeppelin Contracts library can be downloaded by running:

$ npm install --save-dev @openzeppelin/contracts
You should always use the library from these published releases: copy-pasting library source code into your project is a dangerous practice that makes it very easy to introduce security vulnerabilities in your contracts.
To use one of the OpenZeppelin Contracts, import it by prefixing its path with @openzeppelin/contracts. For example, in order to replace our own Auth contract, we will import @openzeppelin/contracts/access/Ownable.sol to add access control to Box:

// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// Import Ownable from the OpenZeppelin Contracts library
import "@openzeppelin/contracts/access/Ownable.sol";

// Make Box inherit from the Ownable contract
contract Box is Ownable {
    uint256 private _value;

    event ValueChanged(uint256 value);

    // The onlyOwner modifier restricts who can call the store function
    function store(uint256 value) public onlyOwner {
        _value = value;
        emit ValueChanged(value);
    }

    function retrieve() public view returns (uint256) {
        return _value;
    }
}
The OpenZeppelin Contracts documentation is a great place to learn about developing secure smart contract systems. It features both guides and a detailed API reference: see for example the Access Control guide to know more about the Ownable contract used in the code sample above.

Deploying and interacting with smart contracts
Unlike most software, smart contracts don’t run on your computer or somebody’s server: they live on the Ethereum network itself. This means that interacting with them is a bit different from more traditional applications.

This guide will cover all you need to know to get you started using your contracts, including:

Setting up a Local Blockchain

Deploying a Smart Contract

Interacting from the Console

Interacting Programmatically

Instructions are available for both Truffle and Hardhat. Choose your preference using this toggle!

Toggle Hardhat or Truffle
Setting up a Local Blockchain
Before we begin, we first need an environment where we can deploy our contracts. The Ethereum blockchain (often called "mainnet", for "main network") requires spending real money to use it, in the form of Ether (its native currency). This makes it a poor choice when trying out new ideas or tools.

To solve this, a number of "testnets" (for "test networks") exist: these include the Ropsten, Rinkeby, Kovan and Goerli blockchains. They work very similarly to mainnet, with one difference: you can get Ether for these networks for free, so that using them doesn’t cost you a single cent. However, you will still need to deal with private key management, blocktimes in the range of 5 to 20 seconds, and actually getting this free Ether.

During development, it is a better idea to instead use a local blockchain. It runs on your machine, requires no Internet access, provides you with all the Ether that you need, and mines blocks instantly. These reasons also make local blockchains a great fit for automated tests.

If you want to learn how to deploy and use contracts on a public blockchain, like the Ethereum testnets, head to our Connecting to Public Test Networks guide.
Hardhat comes with a local blockchain built-in, the Hardhat Network.

Upon startup, Hardhat Network will create a set of unlocked accounts and give them Ether.

$ npx hardhat node
Hardhat Network will print out its address, http://127.0.0.1:8545, along with a list of available accounts and their private keys.

Keep in mind that every time you run Hardhat Network, it will create a brand new local blockchain - the state of previous runs is not preserved. This is fine for short-lived experiments, but it means that you will need to have a window open running Hardhat Network for the duration of these guides.

Hardhat will always spin up an instance of Hardhat Network when no network is specified and there is no default network configured or the default network is set to hardhat.
You can also run an actual Ethereum node in development mode. These are a bit more complex to set up, and not as flexible for testing and development, but are more representative of the real network.
Deploying a Smart Contract
In the Developing Smart Contracts guide we set up our development environment.

If you don’t already have this setup, please create and setup the project and then create and compile our Box smart contract.

With our project setup complete we’re now ready to deploy a contract. We’ll be deploying Box, from the Developing Smart Contracts guide. Make sure you have a copy of Box in contracts/Box.sol.

Hardhat doesn’t currently have a native deployment system, instead we use scripts to deploy contracts.

We will create a script to deploy our Box contract. We will save this file as scripts/deploy.js.

// scripts/deploy.js
async function main () {
  // We get the contract to deploy
  const Box = await ethers.getContractFactory('Box');
  console.log('Deploying Box...');
  const box = await Box.deploy();
  await box.deployed();
  console.log('Box deployed to:', box.address);
}

main()
  .then(() => process.exit(0))
  .catch(error => {
    console.error(error);
    process.exit(1);
  });
We use ethers in our script, so we need to install it and the @nomiclabs/hardhat-ethers plugin.

$ npm install --save-dev @nomiclabs/hardhat-ethers ethers
We need to add in our configuration that we are using the @nomiclabs/hardhat-ethers plugin.

// hardhat.config.js
require('@nomiclabs/hardhat-ethers');

...
module.exports = {
...
};
Using the run command, we can deploy the Box contract to the local network (Hardhat Network):

$ npx hardhat run --network localhost scripts/deploy.js
Deploying Box...
Box deployed to: 0x5FbDB2315678afecb367f032d93F642f64180aa3
Hardhat doesn’t keep track of your deployed contracts. We displayed the deployed address in our script (in our example, 0x5FbDB2315678afecb367f032d93F642f64180aa3). This will be useful when interacting with them programmatically.
All done! On a real network this process would’ve taken a couple of seconds, but it is near instant on local blockchains.

If you got a connection error, make sure you are running a local blockchain in another terminal.
Remember that local blockchains do not persist their state throughout multiple runs! If you close your local blockchain process, you’ll have to re-deploy your contracts.
Interacting from the Console
With our Box contract deployed, we can start using it right away.

We will use the Hardhat console to interact with our deployed Box contract on our localhost network.

We need to specify the address of our Box contract we displayed in our deploy script.
It’s important that we explicitly set the network for Hardhat to connect our console session to. If we don’t, Hardhat will default to using a new ephemeral network, which our Box contract wouldn’t be deployed to.
$ npx hardhat console --network localhost
Welcome to Node.js v12.22.1.
Type ".help" for more information. 
> const Box = await ethers.getContractFactory('Box');
undefined
> const box = await Box.attach('0x5FbDB2315678afecb367f032d93F642f64180aa3')
undefined
Sending transactions
Box's first function, store, receives an integer value and stores it in the contract storage. Because this function modifies the blockchain state, we need to send a transaction to the contract to execute it.

We will send a transaction to call the store function with a numeric value:

> await box.store(42)
{
  hash: '0x3d86c5c2c8a9f31bedb5859efa22d2d39a5ea049255628727207bc2856cce0d3',
...
Querying state
Box's other function is called retrieve, and it returns the integer value stored in the contract. This is a query of blockchain state, so we don’t need to send a transaction:

> await box.retrieve()
BigNumber { _hex: '0x2a', _isBigNumber: true }
Because queries only read state and don’t send a transaction, there is no transaction hash to report. This also means that using queries doesn’t cost any Ether, and can be used for free on any network.

Our Box contract returns uint256 which is too large a number for JavaScript so instead we get returned a big number object. We can display the big number as a string using (await box.retrieve()).toString().
> (await box.retrieve()).toString()
'42'
To learn more about using the console, check out the Hardhat documentation.

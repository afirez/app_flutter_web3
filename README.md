# app_flutter_web3

A new Flutter project.

## Getting Started

This project is a starting point for a Flutter application.

A few resources to get you started if this is your first Flutter project:

- [Lab: Write your first Flutter app](https://flutter.dev/docs/get-started/codelab)
- [Cookbook: Useful Flutter samples](https://flutter.dev/docs/cookbook)

For help getting started with Flutter, view our
[online documentation](https://flutter.dev/docs), which offers tutorials,
samples, guidance on mobile development, and a full API reference.

## Install

https://ethereum.org/zh/developers/docs/programming-languages/dart/

https://www.geeksforgeeks.org/flutter-and-blockchain-hello-world-dapp/

```bash
# node -v 
# v14.19.3
# npm -v 
# 6.14.17
# https://github.com/nvm-sh/nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
nvm install 14
nvm use 14
npm install -g truffle

flutter create -i swift -a kotlin --org com.web3.dapp app_flutter_web3

cd app_flutter_web3 && truffle init

```

## create contract 

```bash
truffle create contract HelloWorld
```

```solidity
# 添加变量
string public yourName ;
```

```solidity
# 在构造函数中初始化变量
  constructor() public {
    yourName = "Unknown" ;
  }
```

```solidity
# 声明函数
function setName(string memory nm) public{
        yourName = nm ;
}
```

## Compiling and Migrating 

```bash
truffle compile

# Compiling your contracts...
# ===========================
# ✓ Fetching solc version list from solc-bin. Attempt #1
# ✓ Downloading compiler. Attempt #1.
# > Compiling ./contracts/HelloWorld.sol
# > Compiling ./contracts/Migrations.sol
# > Compilation warnings encountered:

#     Warning: Visibility for constructor is ignored. If you want the contract to be non-deployable, making it "abstract" is sufficient.
#  --> project:/contracts/HelloWorld.sol:7:3:
#   |
# 7 |   constructor() public {
#   |   ^ (Relevant source part starts here and spans across multiple lines).


# > Artifacts written to /Users/afirez/studio/flutter/app_flutter_web3/build/contracts
# > Compiled successfully using:
#    - solc: 0.8.15+commit.e14f2714.Emscripten.clang
```

### Migration

You’ll see one JavaScript file already in the migrations/ directory: 1_initial_migration.js. This handles deploying the Migrations.sol contract to observe subsequent smart contract migrations, and ensures we don’t double-migrate unchanged contracts in the future.

Let’s create our own migration script :

Create a new file named 2_deploy_contracts.js in the migrations/ directory.

Add the following content to the 2_deploy_contracts.js file:

```js
const HelloWorld = artifacts.require("HelloWorld");
  
module.exports = function (deployer) {
  deployer.deploy(HelloWorld);
};
```

- Before we can migrate our contract to the blockchain, we need to have a blockchain running. For this article, we’re going to use [**Ganache**](https://trufflesuite.com/ganache/), a personal blockchain for Ethereum development you can use to deploy contracts, develop applications, and run tests. If you haven’t already, download Ganache and double-click the icon to launch the application. This will generate a blockchain running locally on port 7545.

- Add the following content to the truffle-config.js file:

```js
module.exports = {
  networks: {
     development: {
      host: "127.0.0.1",     // Localhost (default: none)
      port: 7545,            // Standard Ethereum port (default: none)
      network_id: "*",       // Any network (default: none)
     },
  },
    contracts_build_directory: "./src/artifacts/",
      
  // Configure your compilers
  compilers: {
    solc: {    
      
       // See the solidity docs for advice
       // about optimization and evmVersion
        optimizer: {
          enabled: false,
          runs: 200
        },
        evmVersion: "byzantium"
    }
  }
};
```

- Migrating the contract to the blockchain, run:

```bash
truffle migrate

# Compiling your contracts...
# ===========================
# > Compiling ./contracts/HelloWorld.sol
# > Compilation warnings encountered:

#     Warning: Visibility for constructor is ignored. If you want the contract to be non-deployable, making it "abstract" is sufficient.
#  --> project:/contracts/HelloWorld.sol:7:3:
#   |
# 7 |   constructor() public {
#   |   ^ (Relevant source part starts here and spans across multiple lines).


# > Artifacts written to /Users/afirez/studio/flutter/app_flutter_web3/build/contracts
# > Compiled successfully using:
#    - solc: 0.8.15+commit.e14f2714.Emscripten.clang


# Starting migrations...
# ======================
# > Network name:    'ganache'
# > Network id:      5777
# > Block gas limit: 6721975 (0x6691b7)
# ...
```

## Testing the Smart Contract

In Truffle, we can write tests either in JavaScript or Solidity, In this article, we’ll be writing our tests in Javascript using the Chai and Mocha libraries.

Create a new file named helloWorld.js in the test/ directory.
Add the following content to the helloWorld.js file:

```js
const HelloWorld = artifacts.require("HelloWorld") ;
  
contract("HelloWorld" , () => {
    it("Hello World Testing" , async () => {
       const helloWorld = await HelloWorld.deployed() ;
       await helloWorld.setName("User Name") ;
       const result = await helloWorld.yourName() ;
       assert(result === "User Name") ;
    });
});
```

- HelloWorld: The smart contract we want to test, We begin our test by importing our HelloWorld contract using artifacts.require.
- To test the setName function, recall that it accepts a name(string) as an argument.
- Also, the yourName variable in our contract is using public modifier, which we can use as a getter from the outside function.
- Truffle imports Chai so we can use the assert function. We pass the actual value and the expected value, To check that the name is set properly or not, assert(result === “User Name”) ;.

### Running the tests

- Running the test as:

```bash
truffle test
# If all the test pass, you’ll see the console output similar to this:
#
# Using network 'test'.
#
# Compiling your contracts...
# ===========================
# > Everything is up to date, there is nothing to compile.
#
#   Contract: HelloWorld
#     ✔ Hello World Testing (1078ms)
#
#   1 passing (1s)

```

## Contract linking with Flutter

- In the pubspec.yaml file import the following packages :

```yaml
provider: ^4.3.3
web3dart: ^1.2.3
http: ^0.12.2
web_socket_channel: ^1.2.0
```

- Also, add the asset build/contracts/HelloWorld.json to pubspec.yaml file which is generated by truffle-config.js while we migrate our contract.

```yaml
assets:
    - build/contracts/HelloWorld.json
```

1. Create a new file named contract_linking.dart in the lib/ directory.
2. Add the following content to the file:

```dart

import 'package:flutter/material.dart';
  
class ContractLinking extends ChangeNotifier {
    
}
```

### Variables

Add the following variable on the next line after class ContractLinking extends ChangeNotifier {.

```dart
final String _rpcUrl = "http://10.0.2.2:7545";
final String _wsUrl = "ws://10.0.2.2:7545/";
final String _privateKey = "Enter Private Key";
```

- The library web3dart won’t send signed transactions to miners itself. Instead, it relies on an RPC client to do that. For the WebSocket URL just modify the RPC URL. You can get the RPC URL from the ganache :

RPC SERVER: 

http://127.0.0.1:7545

- Get the Private Key from ganache:

PRIVATEKEY:

- Declare the following variables below :

```dart

  Web3Client _client;
  bool isLoading = true;
  
  String _abiCode;
  EthereumAddress  _contractAddress;
  
  Credentials _credentials;
  
  DeployedContract _contract;
  ContractFunction _yourName;
  ContractFunction _setName;
  
  String deployedName;
```

1. **_client** variable will be used to establish a connection to the ethereum rpc node with the help of WebSocket.
2. **isLoading** variable will be used to check the state of the contract.
3. **_abiCode** variable will be used to, read the contract abi.
4. **_contractAddress** variable will be used to store the contract address of the deployed smart contract.
5. **_credentials** variable will store the credentials of the smart contract deployer.
6. **_contract** variable will be used to tell Web3dart where our contract is declared.
7. **_yourName** and **_setName** variable will be used to store the functions declared in our HelloWorld.sol smart contract.
8. **deployedName** will hold the name from the smart contract.

### Functions

After declaring the above variable’s, Declare the following functions below it :

```dart
ContractLinking() {
    initialSetup();
  }
  
  initialSetup() async {
      
    // establish a connection to the ethereum rpc node. The socketConnector
    // property allows more efficient event streams over websocket instead of
    // http-polls. However, the socketConnector property is experimental.
    _client = Web3Client(_rpcUrl, Client(), socketConnector: () {
      return IOWebSocketChannel.connect(_wsUrl).cast<String>();
    });
  
    await getAbi();
    await getCredentials();
    await getDeployedContract();
  }
  
  Future<void> getAbi() async {
      
    // Reading the contract abi
    String abiStringFile =
        await rootBundle.loadString("src/artifacts/HelloWorld.json");
    var jsonAbi = jsonDecode(abiStringFile);
    _abiCode = jsonEncode(jsonAbi["abi"]);
  
    _contractAddress =
        EthereumAddress.fromHex(jsonAbi["networks"]["5777"]["address"]);
  }
  
  Future<void> getCredentials() async {
    _credentials = await _client.credentialsFromPrivateKey(_privateKey);
  }
  
  Future<void> getDeployedContract() async {
      
    // Telling Web3dart where our contract is declared.
    _contract = DeployedContract(
        ContractAbi.fromJson(_abiCode, "HelloWorld"), _contractAddress);
  
    // Extracting the functions, declared in contract.
    _yourName = _contract.function("yourName");
    _setName = _contract.function("setName");
    getName();
  }
  
  getName() async {
      
    // Getting the current name declared in the smart contract.
    var currentName = await _client
        .call(contract: _contract, function: _yourName, params: []);
    deployedName = currentName[0];
    isLoading = false;
    notifyListeners();
  }
  
  setName(String nameToSet) async {
      
    // Setting the name to nameToSet(name defined by user)
    isLoading = true;
    notifyListeners();
    await _client.sendTransaction(
        _credentials,
        Transaction.callContract(
            contract: _contract, function: _setName, parameters: [nameToSet]));
    getName();
  }
```

### Creating a UI to interact with the smart contract

1. Create a new file named helloUI.dart in the lib/ directory.
2. Add the following content to the file:

```dart

import 'package:flutter/material.dart';
import 'package:app_flutter_web3/contract_linking.dart';
import 'package:provider/provider.dart';
  
class HelloUI extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
      
    // Getting the value and object or contract_linking
    var contractLink = Provider.of<ContractLinking>(context);
  
    TextEditingController yourNameController = TextEditingController();
  
    return Scaffold(
      appBar: AppBar(
        title: Text("Hello World !"),
        centerTitle: true,
      ),
      body: Container(
        padding: EdgeInsets.symmetric(horizontal: 20),
        child: Center(
          child: contractLink.isLoading
              ? CircularProgressIndicator()
              : SingleChildScrollView(
            child: Form(
              child: Column(
                children: [
                  Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Text(
                        "Hello ",
                        style: TextStyle(
                            fontWeight: FontWeight.bold, fontSize: 52),
                      ),
                      Text(
                        contractLink.deployedName,
                        style: TextStyle(
                            fontWeight: FontWeight.bold,
                            fontSize: 52,
                            color: Colors.tealAccent),
                      ),
                    ],
                  ),
                  Padding(
                    padding: EdgeInsets.only(top: 29),
                    child: TextFormField(
                      controller: yourNameController,
                      decoration: InputDecoration(
                          border: OutlineInputBorder(),
                          labelText: "Your Name",
                          hintText: "What is your name ?",
                          icon: Icon(Icons.drive_file_rename_outline)),
                    ),
                  ),
                  Padding(
                    padding: EdgeInsets.only(top: 30),
                    child: ElevatedButton(
                      child: Text(
                        'Set Name',
                        style: TextStyle(fontSize: 30),
                      ),
                      style: ElevatedButton.styleFrom(
                        primary: Colors.green,
                      ),
                      onPressed: () {
                        contractLink.setName(yourNameController.text);
                        yourNameController.clear();
                      },
                    ),
                  )
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

- Update the main.dart as :

```dart
import 'package:flutter/material.dart';
import 'package:app_flutter_web3/contract_linking.dart';
import 'package:app_flutter_web3/helloUI.dart';
import 'package:provider/provider.dart';
  
void main() {
  runApp(MyApp());
}
  
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
      
    // Inserting Provider as a parent of HelloUI()
    return ChangeNotifierProvider<ContractLinking>(
      create: (_) => ContractLinking(),
      child: MaterialApp(
        title: "Hello World",
        theme: ThemeData(
            brightness: Brightness.dark,
            primaryColor: Colors.cyan[400],
            accentColor: Colors.deepOrange[200]),
        home: HelloUI(),
      ),
    );
  }
}
```

### Interacting with the complete Dapp

- Now we’re ready to use our dapp!
- Just RUN the Flutter Project.

```bash
flutter run
```

As you can see the Hello Unknown, in the UI is actually coming from the smart contract, variable yourName .

When you type your name into the TextFormField and Press `Set Name` ElevatedButton, it will invoke the setName function from contract_linking.dart which will directly invoke the setName function of our smart contract (HelloWorld.sol).

Congratulations! You have taken a huge step to become a full-fledged mobile dapp developer. For developing locally, you have all the tools you need to start making more advanced dapps.

If you stuck somewhere, do check out the [GitHub repository](https://github.com/afirez/app_flutter_web3.git) for complete code.










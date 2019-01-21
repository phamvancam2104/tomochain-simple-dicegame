# Building A Dice Game DApp On The TomoChain Blockchain

Exploring interesting use cases for the TomoChain blockchain by building a simple dice roll DApp game with Truffle Framework.


`DApp` is an abbreviation for Decentralized app. DApps are a new paradigm for building apps where a back end centralized server is replaced by a decentralized peer to peer network.

Industry-wide, we’re just beginning to scratch the surface of potential blockchain use cases. Most people associate ‘blockchain’ with ‘cryptocurrencies,’ but new use cases for blockchain technology are emerging everyday.

Today, I’ll show you how to build a simple dice game with TomoChain testnet as a means to explore different and interesting uses cases for blockchain technology.

## What is a DApp?
DApps built on blockchain technology, where platforms such as TomoChain serve as the backend for the application.

One of the most popular DApps, cryptokitties, is collectibles-style game built on Ethereum. When we build a game with TomoChain or Ethereum, essentially, each game action and transaction is stored on the used blockchain.

## Roll the Dice DApp
Let’s create a simple dice roll game.

`Player` will roll the dice and chance to win 10 ether. ‘Target’ will be set between 1 to 6.

You can try our Roll the Dice here. 

Note: Game will be deployed on TomoChain testnet network only.

Now let’s dive into the code.

## How To Play The Game
* Click on “Get new bet” to get the target.
* Next, roll the Dice by clicking “Roll it”.
* If you get the same number, 10 ether will be transfer to your account.

![gamedice](/assets/ex.png)

## Prerequisite

* Understanding of Nodejs
* Basic understanding of Smart contract
* Truffle framework
* Understanding of Html and Javascript
* Metamask wallet
* Dice smart contract

Our Dice.sol smart contract will control the core logic for our game. Let’s have a look at the code:

````
struct Bet{
  uint8  currentBet; // this is target
  bool  isBetSet; //default value is false 
  uint8  destiny; // 
 }
mapping(address => Bet) private bets;
uint8 private randomFactor;
````

We will define Bet structure as having 3 variables:

* `currentBet`: This is used to set a new bet.
* `isBetSet`: To check if the bet is set or not?
* `destiny`: Contains the number when you roll the dice.

We will also create mapping which will track the bets of players, and add a randomFactor, which will be used to randomize our results.

Events: We have 2 events, one will emit when the bet is set for a player and other is for the game result. Events help in conveying state change on the frontend.

````
event NewBetIsSet(address bidder , uint8 currentBet);
event GameResult(address bidder, uint8 currentBet , uint8 destiny);
````
### Getting a new bet
To take a bet from a new player, we will perform the following steps:

* Check if a bet is already set for the player
* Mark the bet as set now
* Get the random number and set the current bet
* Emit `NewBetIsSet` event and return the current bet

````
function getNewbet() public returns(uint8){
  require(bets[msg.sender].isBetSet == false);
  bets[msg.sender].isBetSet = true;
  bets[msg.sender].currentBet = random();
  randomFactor += bets[msg.sender].currentBet;
  emit NewBetIsSet(msg.sender, bets[msg.sender].currentBet);
  return bets[msg.sender].currentBet;
 }
````
### Rolling the Dice
Next, we will roll the dice. For this, we will perform the following steps:

* Check if a bet is set for the player
* Get the random number and set the destiny
* Mark the isBetSet variable false
* Check if currentbet and destiny is equal, if yes transfer the prize(0.00001 ether) to the player and emit the GameResult event.
* Else only emit the GameResult event if currentBet and destiny is not equal.

````
function roll() public returns(address , uint8 , uint8){
  require(bets[msg.sender].isBetSet == true);
  bets[msg.sender].destiny = random();
  randomFactor += bets[msg.sender].destiny;
  bets[msg.sender].isBetSet = false;
  if(bets[msg.sender].destiny == bets[msg.sender].currentBet){
   msg.sender.transfer(100000000000000);
   emit GameResult(msg.sender, bets[msg.sender].currentBet, bets[msg.sender].destiny);   
  }else{
   emit GameResult(msg.sender, bets[msg.sender].currentBet, bets[msg.sender].destiny);
  }
  return (msg.sender , bets[msg.sender].currentBet , bets[msg.sender].destiny);
 }
````
We are adding destiny and currentBet to our randomFactor every time. This helps us randomize our bets efficiently.

Other than above core functions, we have an isBetSet function to tell if the bet is set for a player and a random function to get random numbers for our dice.

````
function isBetSet() public view returns(bool){
  return bets[msg.sender].isBetSet;
 }
function random() private view returns (uint8) {
 uint256 blockValue = uint256(blockhash(block.number-1 +    block.timestamp));
        blockValue = blockValue + uint256(randomFactor);
        return uint8(blockValue % 5) + 1;
 }
````

Fallback Function: We will also add a fallback function. This function is executed if a contract is called and no other function matches the specified function identifier, or if no data is supplied. These functions are also executed whenever a contract would receive plain Ether, without any data.

````
function() public payable{}
````
Note: You should test your smart contract. You can learn more about testing smart contracts in some of our previous tutorials.

### Truffle framework
We will use Truffle framework to develop our DApp. Truffle has prebuilt packages which they call boxes. Truffle boxes help in getting the boilerplate code to develop a DApp. You can check more about truffle boxes here. We will use basic pet-box which will give us the boilerplate code for Dice game DApp. You can learn more about truffle pet-box here.

Let’s walk through app.js and understand what's happening on the frontend.

We will define our app variable and declare variables we will use throughout app.js.

We will also add an ‘init’ function in which will initialize web3 provider. Web3 provider allows your application to communicate with an TomoChain Node.

MetaMask, a chrome extension wallet, will inject web3.js. Here, we will see if any web3 provider already exists. If not, it will try to connect a local blockchain. For testing purposes, we will run ganache on our local machine and connect to it.

````
App = {
  web3Provider: null,
  contracts: {},
  account: '0x0',
  hasVoted: false,
init: function() {
    return App.initWeb3();
  },
initWeb3: function() {
    if (typeof web3 !== 'undefined') {
      // If a web3 instance is already provided by Meta Mask.
      App.web3Provider = web3.currentProvider;
      web3 = new Web3(web3.currentProvider);
    } else {
      App.web3Provider = new Web3.providers.HttpProvider('http://localhost:8545');
      web3 = new Web3(App.web3Provider);
    }
    return App.initContract();
  },
````
### Getting Dice Contract
Truffle petbox gives us truffle-contract.js which gives us the boilerplate code to interact with the contract. We use ABI (Application Programming Interface), a JSON representation of our contractl to interact with our contract on the frontend.

If we don’t use truffle we manually need to change this ABI every time when change and compile our contract. Whenever you compile a solidity smart contract, it will generate a JSON file. This JSON file is ABI to interact with the smart contract.

````
initContract: function() {
    $.getJSON("Dice.json", function(dice) {
      // Instantiate a new truffle contract from the artifact
      App.contracts.Dice = TruffleContract(dice);
      // Connect provider to interact with contract
      App.contracts.Dice.setProvider(App.web3Provider);
App.listenForEvents();
return App.render();
    });
  },
````
We now get the Dice.json file, which is JSON representation of our smart contract. We initiate our Dice contract and set the web3.provider. truffle-contract.js are helping us here by providing TruffleContract function. You can check Dice.json and truffle-contract.js for more details.

Listening Events
Events are a crucial part of any DApp.

Asynchronous and blockchain transactions take time. Events help us in tracking the status inside the DApp.

We will change our interface to show changes to users. We are listening to both events GameResult and NewBetIsSet and passing event object to render UI accordingly.

````
listenForEvents: function() {
    App.contracts.Dice.deployed().then(function(instance) {
      instance.GameResult({}, {}).watch(function(error, event) {
        console.log("event triggered", event)
        // Reload when a new vote is recorded
        App.render(event);
      });
    instance.NewBetIsSet({}, {}).watch(function(error, event) {
        console.log("event triggered", event)
        // Reload when a new vote is recorded
        App.render(event);
      });
});
  },
````  
Check render method. We are checking events and showing results to users accordingly and also we are calling isBetSet method to know if a bet is set for a user or not.

````
render: function(event) {
    var gameInstance;
    // Load account data
    web3.eth.getCoinbase(function(err, account) {
      if (err === null) {
        App.account = account;
        $("#accountAddress").html("Your Account : " + account );
      }
    });
if(event.event == "NewBetIsSet"){
    $("#newBet")
      .text("Your target is : " + event.args.currentBet.toNumber());
    }
if(event.event == "GameResult") {
        var destiny = event.args.destiny.toNumber();
      var currentBet = event.args.currentBet.toNumber();
      var doWeHaveAWinner = (destiny == currentBet);
      if(doWeHaveAWinner){
          $("#result").text("we have a winner");
      }else{
          $("#result").text("Sorry bad luck, your got " + destiny);
      }
    }
// Load contract data
    App.contracts.Dice.deployed().then(function(instance) {
      gameInstance = instance;
      return gameInstance.isBetSet();
    }).then(function(isBetSet) {
      var message = $("#message");
      if(isBetSet){
        message.text('Bet is Set, Roll the Dice')
      }else{
      message.text('Set New Bet');
    }
    }).catch(function(error) {
      console.warn(error);
    });
  }
````
Also, we are defining roll and getNewBet method which we are calling from index.html on button clicks.

````
roll : function(){
    App.contracts.Dice.deployed().then(function(instance) {
      return instance.roll({ from: App.account });
    }).then(function(result) {
    }).catch(function(err) {
      console.error(err);
    });
  },
getNewBet: function() {
    $("#result").text("");
    App.contracts.Dice.deployed().then(function(instance) {
      return instance.getNewbet({ from: App.account });
    }).then(function(result) {
       }).catch(function(err) {
      console.error(err);
    });
  }
````
Deploying our Dapp
We will use ganache for deploying our DApp locally. If you don’t have ganache, you can download it here. Run the commands below to deploy our contracts.

````
truffle compile
truffle migrate --reset
````
This will deploy our smart contract to interact with the smart contract you can use truffle console command.

### Improvement
There are a lot of improvements which can be made to our Dice apart from UI.

Here are a few suggested improvements which you add to your version of the Game.

* Optimize the memory. There is no need to story destiny, you can remove that.
* Optimize the amount of Gas we are using.
* Add a withdraw function to take out the extra ether from the contract.
* Make random function better. Generating random numbers on blockchains are itself a challenge as everything on blockchain is public.
* Improve the UI. Make it roll 😃

## Conclusion
DApps are a new paradigm to for building applications on the internet, and we’re just scratching the surface. Instead of hosting an app on Heroku, we can host our app on IPFS (decentralize peer to peer file system).

DApps decentralize the way we interact on the internet. DApps run on decentralized networks, in our case TomoChain blockchain, but not every DApp needs to be built with a blockchain.

In the future, you will see more Dapps with awesome UX and better use cases. Now’s the time to explore!
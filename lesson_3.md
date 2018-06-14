# Advanced Solidity Concepts

This lesson will be a bit less flashy, but we’ll learn some really important concepts that will take us closer to building real DApps — things like **contract ownership**, **gas costs**, **code optimization**, and **security**.

##  External Dependencies 

In Lesson 2, we hard-coded the CryptoKitties contract address into our DApp. But what would happen if the CryptoKitties contract had a bug and someone destroyed all the kitties?

For this reason, it often makes sense to have functions that will allow you to update key portions of the DApp.

For example, instead of hard coding the CryptoKitties contract address into our DApp, we should probably have a setKittyContractAddress function that lets us change this address in the future in case something happens to the CryptoKitties contract. So let's do that right now:
```
pragma solidity ^0.4.19;

import "./zombiefactory.sol";

contract KittyInterface {
  function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
  );
}

contract ZombieFeeding is ZombieFactory {

  KittyInterface kittyContract;

  function setKittyContractAddress(address _address) external {
    kittyConract = KittyInterface(_address);    
  }

  function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    if (keccak256(_species) == keccak256("kitty")) {
      newDna = newDna - newDna % 100 + 99;
    }
    _createZombie("NoName", newDna);
  }

  function feedOnKitty(uint _zombieId, uint _kittyId) public {
    uint kittyDna;
    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
    feedAndMultiply(_zombieId, kittyDna, "kitty");
  }

}
```

## Ownable Contracts

There's a problem with our previous code. `setKittyContracts` is `external`, so anyone can call it! That means anyone who called the function could change the address of the CryptoKitties contract, and break our app for all its users. The solution to this is making `setKittyContracts` `Ownable` instead of `external`.

### OpenZeppelin's `Ownable` contract

OpenZeppelin is a library of secure and community-vetted smart contracts that you can use in your own DApps.Below is the Ownable contract taken from their Solidity library: 
```
/**
 * @title Ownable
 * @dev The Ownable contract has an owner address, and provides basic authorization control
 * functions, this simplifies the implementation of "user permissions".
 */
contract Ownable {
  address public owner;
  event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

  /**
   * @dev The Ownable constructor sets the original `owner` of the contract to the sender
   * account.
   */
  function Ownable() public {
    owner = msg.sender;
  }

  /**
   * @dev Throws if called by any account other than the owner.
   */
  modifier onlyOwner() {
    require(msg.sender == owner);
    _;
  }

  /**
   * @dev Allows the current owner to transfer control of the contract to a newOwner.
   * @param newOwner The address to transfer ownership to.
   */
  function transferOwnership(address newOwner) public onlyOwner {
    require(newOwner != address(0));
    OwnershipTransferred(owner, newOwner);
    owner = newOwner;
  }
}

```

So the Ownable contract basically does the following:

- When a contract is created, its constructor sets the `owner` to `msg.sender` (the person who deployed it)

- It adds an `onlyOwner` modifier, which can restrict access to certain functions to only the `owner`

- It allows you to transfer the contract to a new `owner`

`onlyOwner` is such a common requirement for contracts that most Solidity DApps start with a copy/paste of this `Ownable` contract, and then their first contract inherits from it.

## `onlyOwner` Function Modifier

Now that our base contract `ZombieFactory` inherits from `Ownable`, we can use the `onlyOwner` function modifier in `ZombieFeeding` as well.

### Function Modifiers

A function modifier looks just like a function, but uses the keyword `modifier` instead of the keyword `function`. And it can't be called directly like a function can — instead we can attach the modifier's name at the end of a function definition to change that function's behavior.

Let's take a closer look at `onlyOwner`: 
```
/**
 * @dev Throws if called by any account other than the owner.
 */
modifier onlyOwner() {
  require(msg.sender == owner);
  _;
}
```
Here's how we would apply it to one of our functions: 

```
 function setKittyContractAddress(address _address) external onlyOwner {
    kittyContract = KittyInterface(_address);
  }
```


In the case of `onlyOwner`, adding this modifier to a function makes it so only the owner of the contract (you, if you deployed it) can call that function.

## Struct Packing to Save Gas

In Lesson 1, we mentioned that there are other types of `uint`s: `uint8`, `uint16`, `uint32`, etc.

Normally there's no benefit to using these sub-types because Solidity reserves 256 bits of storage regardless of the `uint` size. For example, using `uint8` instead of `uint` (`uint256`) won't save you any gas.

But there's an exception to this: inside `struct`s.

If you have multiple `uint`s inside a struct, using a smaller-sized `uint` when possible will allow Solidity to pack these variables together to take up less storage.

For example: 
```
struct NormalStruct {
  uint a;
  uint b;
  uint c;
}

struct MiniMe {
  uint32 a;
  uint32 b;
  uint c;
}

// `mini` will cost less gas than `normal` because of struct packing
NormalStruct normal = NormalStruct(10, 20, 30);
MiniMe mini = MiniMe(10, 20, 30);
```

For this reason, inside a `struct` you'll want to use the smallest integer sub-types you can get away with.

You'll also want to cluster identical data types together (i.e. put them next to each other in the `struct`) so that Solidity can minimize the required storage space. For example, a `struct` with fields `uint c; uint32 a; uint32 b;` will cost less gas than a struct with fields `uint32 a; uint c; uint32 b;` because the `uint32` fields are clustered together.

## `level` and `readyTime`

Let's add two new properties to our zombies! The `level` property is pretty self-explanatory. Later on, when we create a battle system, zombies who win more battles will level up over time and get access to more abilities.

The `readyTime` property requires a bit more explanation. The goal is to add a "cooldown period", an amount of time a zombie has to wait after feeding or attacking before it's allowed to feed / attack again. Without this, the zombie could attack and multiply 1,000 times per day, which would make the game way too easy.

In order to keep track of how much time a zombie has to wait until it can attack again, we can use Solidity's time units.

### Time units

Solidity provides some native units for dealing with time.

The variable `now` will return the current unix timestamp (the number of seconds that have passed since January 1st 1970). The unix time as I write this is `1515527488`.

Solidity also contains the time units `seconds`, `minutes`, `hours`, `days`, `weeks` and `years`. These will convert to a `uint` of the number of seconds in that length of time. So `1 minutes` is `60`, `1 hours` is `3600` (60 seconds x 60 minutes), `1 days` is `86400` (24 hours x 60 minutes x 60 seconds), etc.

We can use these units to implement a cooldown timer in `feedAndMultiply` such that:
1. Feeding triggers a zombie's cooldown, and
2. Zombies can't feed on kitties until their cooldown period has passed

This will make it so zombies can't just feed on unlimited kitties and multiply all day. In the future when we add battle functionality, we'll make it so attacking other zombies also relies on the cooldown.

```
pragma solidity ^0.4.19;

import "./zombiefactory.sol";

contract KittyInterface {
  function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
  );
}

contract ZombieFeeding is ZombieFactory {

  KittyInterface kittyContract;

  function setKittyContractAddress(address _address) external onlyOwner {
    kittyContract = KittyInterface(_address);
  }

  // 1. Define `_triggerCooldown` function here
  function _triggerCooldown(Zombie storage _zombie) internal {
    _zombie.readyTime = uint32(now + cooldownTime);
  }
  // 2. Define `_isReady` function here
  function _isReady(Zombie storage _zombie) internal view returns (bool) {
    return (_zombie.readyTime <= now);
  }

  function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    if (keccak256(_species) == keccak256("kitty")) {
      newDna = newDna - newDna % 100 + 99;
    }
    _createZombie("NoName", newDna);
  }

  function feedOnKitty(uint _zombieId, uint _kittyId) public {
    uint kittyDna;
    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
    feedAndMultiply(_zombieId, kittyDna, "kitty");
  }

}
```

## Public Functions & Security

Now let's modify `feedAndMultiply` to take our cooldown timer into account. Looking back at this function, you can see we made it  made it `public` in the previous lesson. An important security practice is to examine all your `public` and `external` functions, and try to think of ways users might abuse them. Remember — unless these functions have a modifier like `onlyOwner`, any user can call them and pass them any data they want to.

Re-examing this particular function, the user could call the function directly and pass in any `_targetDna` or `_species` they want to. This doesn't very game-like — we want them to follow our rules!

On closer inspection, this function only needs to be called by `feedOnKitty()`, so the easiest way to prevent these exploits is to make it `internal`

So let's do that:
```
  // 1. Make this function internal
  function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) internal {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    // 2. Add a check for `_isReady` here
    require(_isReady(myZombie));
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    if (keccak256(_species) == keccak256("kitty")) {
      newDna = newDna - newDna % 100 + 99;
    }
    _createZombie("NoName", newDna);
    // 3. Call `triggerCooldown`
    _triggerCooldown(myZombie);
  }
 ``` 
## Helper Methods + More Function Modifiers 

Great! Our zombie now has a functional cooldown timer.

Next, we're going to add some additional helper methods. Let's create a new file called `zombiehelper.sol`, which imports `zombiefeeding.sol`. This will help to keep our code organized.

Let's make it so zombies gain special abilities after reaching a certain level. But in order to do that, first we'll need to learn a little bit more about function modifiers.

### Function Modifiers with Arguments 

Previously we looked at the simple example of `onlyOwner`. But function modifiers can also take arguments. Let's try making our own modifier that uses the zombie level property to restrict access to special abilities.

```
pragma solidity ^0.4.19;

import "./zombiefeeding.sol";

contract ZombieHelper is ZombieFeeding {

  // Start here
  modifier aboveLevel(uint _level, uint _zombieId) {
    require(zombies[_zombieId].level >= _level);
    _;
  }  
}
```

## Zombie Modifiers 

Now let's use our aboveLevel modifier to create some functions.

Our game will have some incentives for people to level up their zombies:

- For zombies level 2 and higher, users will be able to change their name.
- For zombies level 20 and higher, users will be able to give them custom DNA.

We'll implement these functions below. 

```
pragma solidity ^0.4.19;

import "./zombiefeeding.sol";

contract ZombieHelper is ZombieFeeding {

  modifier aboveLevel(uint _level, uint _zombieId) {
    require(zombies[_zombieId].level >= _level);
    _;
  }

  // Start here
  function changeName(uint _zombieId, string _newName) external aboveLevel(2, _zombieId) {
      require(msg.sender == zombieToOwner[_zombieId]);
      zombies[_zombieId].name = _newName;
  } 

  function changeDna(uint _zombieId, uint _newDna) external aboveLevel(20, _zombieId) {
    require(msg.sender == zombieToOwner[_zombieId]);
    zombies[_zombieId].dna = _newDna;
  }
}
```

## Saving Gas With 'View' Functions

Let's add one more function: our DApp needs a method to view a user's entire zombie army — let's call it getZombiesByOwner.

This function will only need to read data from the blockchain, so we can make it a `view` function. Which brings us to an important topic when talking about gas optimization:

### View functions don't cost gas

`view` functions don't cost any gas when they're called externally by a user.

This is because `view` functions don't actually change anything on the blockchain – they only read the data. So marking a function with view tells `web3.js` that it only needs to query your local Ethereum node to run the function, and it doesn't actually have to create a transaction on the blockchain (which would need to be run on every single node, and cost gas).

We'll cover setting up `web3.js` with your own node later. But for now the big takeaway is that you can optimize your DApp's gas usage for your users by using read-only `external view` functions wherever possible.

> Note: If a `view` function is called internally from another function in the same contract that is not a `view` function, it will still cost gas. This is because the other function creates a transaction on Ethereum, and will still need to be verified from every node. So `view` functions are only free when they're called externally.

### Implementation: user profile 

We're going to implement a function that will return a user's entire zombie army. We can later call this function from web3.js if we want to display a user profile page with their entire army.
```
// Create your function here
  function getZombiesByOwner(address _owner) external view returns(uint[]) {
    
  }
```

## Storage is Expensive 

One of the more expensive operations in Solidity is using `storage` — particularly writes.

This is because every time you write or change a piece of data, it’s written permanently to the blockchain. Forever! Thousands of nodes across the world need to store that data on their hard drives, and this amount of data keeps growing over time as the blockchain grows. So there's a cost to doing that.

In order to keep costs down, you want to avoid writing data to `storage` except when absolutely necessary. Sometimes this involves seemingly inefficient programming logic — like rebuilding an array in `memory` every time a function is called instead of simply saving that array in a variable for quick lookups.

In most programming languages, looping over large data sets is expensive. But in Solidity, this is way cheaper than using `storage` if it's in an `external view` function, since view functions don't cost your users any gas. (And gas costs your users real money!).

### Declaring Arrays in Memory

We can use the `memory` keyword with arrays to create a new array inside a function without needing to write anything to `storage`. The array will only exist until the end of the function call, and this is a lot cheaper gas-wise than updating an array in `storage` — free if it's a `view` function called externally.

Here's how to declare an array in memory:
```
 function getZombiesByOwner(address _owner) external view returns(uint[]) {
    // Start here
    uint[] memory result = new uint[](ownerZombieCount[_owner]);
    return result;
  }
```

> Note: `memory` arrays must be created with a length argument (in this example, `3`). They currently cannot be resized like storage arrays can with `array.push()`, although this may be changed in a future version of Solidity.

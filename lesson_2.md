# Making Zombies Attack Their Victims
![zombies](https://cdn-images-1.medium.com/max/1200/1*-T-fIh5GDl7XvhVAvu6OYA.png)
## Zombie Feeding

In lesson 1, we created a basic data structure to represent a zombie. In this lesson, we'll make the game multiplayer and set up a mechanism to attack and create new zombies.

To accomplish this, we will be creating a new contract called **zombiefeeding.sol**.

## Mappings and Addresses

Solidity has a novel data structure known as a `mapping` that links together two pieces of data. It is very similar to a [hash table](http://www.sparknotes.com/cs/searching/hashtables/section1/). 

Briefly, you can think of it as two arrays of values being mapped on two each other.

> *A more accurate analogy would be to say they're two linked lists being mapped on to each other, but that's a more confusing mental model for me.*

We can use this data structure to associate people's addresses with the zombies they own: 
```
    mapping (uint => address) public zombieToOwner;
    mapping (address => uint) ownerZombieCount;
```

## Addresses 

The ethereum blockchcain is made up of accounts. An account has a balance of ether and can send and receive payments. Each account has a unique ```address```, which looks like this:
```
0x0cE446255506E92DF41614C46F1d6df9Cc969183
```
## Msg.sender

In Solidity, there are certain global variables that are available to all functions. One of these is `msg.sender`. `msg.sender` is kindof like `this`, in that its value depends on the context in which it is used. Generally, `msg.sender` refers to the `address` of the person (or smart contract) that called the function. Here's an example of us using `msg.sender` to update the record of zombies owned by a person:
```
function _createZombie(string _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        zombieToOwner[id] = msg.sender;
        ownerZombieCount[msg.sender]++;
        NewZombie(id, _name, _dna);
    }
```
## Require
We have a function called `_createRandomZombie` that lets users create new zombies. However, if anyone could keep creating unlimited zombies all the time, that wouldn't be very game-like (where's the challenge??). To make things difficult, we can use a cool statement called `require`, that refuses to run the code until a specific condition is met. We're gonna use it to make it so that the `_createRandomZombie_` function can only be called once per player:
```
function sayHiToVitalik(string _name) public returns (string) {
  // Compares if _name equals "Vitalik". Throws an error and exits if not true.
  // (Side note: Solidity doesn't have native string comparison, so we
  // compare their keccak256 hashes to see if the strings are equal)
  require(keccak256(_name) == keccak256("Vitalik"));
  // If it's true, proceed with the function:
  return "Hi!";
}
```

## Inheritance

This one is pretty simple. You can use the keyword `is` to make a function inherit the properties of another function. It's very similar to the concept of interfaces or subclasses. We're going to use it to make the `ZombieFeeding` function (which makes zombies attack people and turn them into zombies) inherit from the `ZombieFactory` function:
```
contract ZombieFeeding is ZombieFactory {

  function feedAndMultiply(uint _zombieId, uint _targetDna) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    _createZombie("NoName", newDna);
  }
}
```

## Storage vs Memory

In Solidity, there are two places where you can store variables: `storage` and `memory`. State variables (variables declared outside funtcions) are by default `storage` and are written permanently into the blockchain, whereas variables declared inside functions are by default `memory` and will disappear when the function call ends.

Usually, you don't have to explicitly declare variables as `storage` or `memory` since it's already being done by the compiler in the background, but in some cases you will have to do this to saev memory or reduce gas.

### `feedAndMultiply()`

It's time to give our zombies the ability to feed and multiply!

When a zombie feeds on some other lifeform, its DNA will combine with the other lifeform's DNA to create a new zombie.

To accomplish this, we will create a function `feedAndMultiply()`, which takes two parameters: `_zombieId`, and `_targetDna`. This function will be `public`. Although our function is public, we don't want to let anyone else feed our zombie through this function. To prevent this, we add a `require` stating that the `msg.sender` must be equal to this zombie's owner.

Once the new zombie has been created, we get its DNA! To do this, we declare a local `Zombie` that we store in the blockchain and add to our `zombies` array. 

The result: 
```
function feedAndMultiply(uint _zombieId, uint _targetDna) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
  }
```
## Zombie DNA

The formula for calculating a new zombie's DNA is simple: It's simply the average between the feeding zombie's DNA and the target's DNA.

For example:
```
function testDnaSplicing() public {
  uint zombieDna = 2222222222222222;
  uint targetDna = 4444444444444444;
  uint newZombieDna = (zombieDna + targetDna) / 2;
  // ^ will be equal to 3333333333333333
}
```
## Internal vs External Functions

In addition to `public` and `private`, Solidity has two more types of visibility for functions, `internal` and `external`.

`internal` is the same as `private`, except that it's also accessible to contracts that inherit from this  contract. 
`external` is similar to `public`, except that these functions can ONLY be called outside the contract — they can't be called by other functions inside the contract. Here's the syntax:
```
contract Sandwich {
  uint private sandwichesEaten = 0;

  function eat() internal {
    sandwichesEaten++;
  }
}

contract BLT is Sandwich {
  uint private baconSandwichesEaten = 0;

  function eatWithBacon() public returns (string) {
    baconSandwichesEaten++;
    // We can call this here because it's internal
    eat();
  }
}
```
## Eating Cryptokitties: How to Interact with other contracts :smirk_cat:

Since both CryptoZombies and CryptoKitties are stored openly on the blockchain, their smart contracts are interoperable!! 

For our contract to talk to another contract on the blockchain that we don't own, we first need to define an **interface**.

i.e. we have to include a basic declaration of the other contract's functions (only the functions that we're going to interact with) in our codebase, so that our contract knows what the other contract's functions look like, and how to interact with them.

Cryptokitties has a function called `getKitty` that we need to interact with, and it looks like this: 
```
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
) {
    Kitty storage kit = kitties[_id];

    // if this variable is 0 then it's not gestating
    isGestating = (kit.siringWithId != 0);
    isReady = (kit.cooldownEndBlock <= block.number);
    cooldownIndex = uint256(kit.cooldownIndex);
    nextActionAt = uint256(kit.cooldownEndBlock);
    siringWithId = uint256(kit.siringWithId);
    birthTime = uint256(kit.birthTime);
    matronId = uint256(kit.matronId);
    sireId = uint256(kit.sireId);
    generation = uint256(kit.generation);
    genes = kit.genes;
}
```

To interact with it, we just add it to our codebase within an inteface contract, like so:
```
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
```

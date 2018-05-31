# Making Zombies Attack Their Victims
![zombies](https://cdn-images-1.medium.com/max/1200/1*-T-fIh5GDl7XvhVAvu6OYA.png)
## Zombie Feeding

In lesson 1, we created a basic data structure to represent a zombie. In this lesson, we'll make the game multiplayer and set up a mechanism to attack and create new zombies. 

## Mappings and Addresses

Solidity has a novel data structure known as a `mapping` that links together two pieces of data. It is very similar to a [hash table](http://www.sparknotes.com/cs/searching/hashtables/section1/). Briefly, you can think of it as two arrays of values being mapped on two each other.

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

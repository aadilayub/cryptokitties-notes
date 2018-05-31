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

### Addresses 

The ethereum blockchcain is made up of accounts. An account has a balance of ether and can send and receive payments. Each account has a unique ```address```, which looks like this:
```
0x0cE446255506E92DF41614C46F1d6df9Cc969183
```
### Msg.sender

In Solidity, there are certain global variables that are available to all functions. One of these is `msg.sender`. `msg.sender` is kindof like `this`, in that its value depends on the context in which it is used. Generally, `msg.sender` refers to the `address` of the person (or smart contract) that called the function. Here's an example of us using `msg.sender` to update the record of zombies owned by a person:
```
function _createZombie(string _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        zombieToOwner[id] = msg.sender;
        ownerZombieCount[msg.sender]++;
        NewZombie(id, _name, _dna);
    }
```

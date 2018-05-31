# Making Zombies Attack Their Victims

## Zombie Feeding

In lesson 1, we created a basic data structure to represent a zombie. In this lesson, we'll make the game multiplayer and set up a mechanism to attack and create new zombies. 

## Mappings and Addresses

Solidity has a novel data structure known as a `mapping` that links together two pieces of data. It is very similar to a [hash table](http://www.sparknotes.com/cs/searching/hashtables/section1/). Briefly, you can think of it as two arrays of values being mapped on two each other.

> *A more accurate analogy would be to say they're two linked lists being mapped on to each other, but that's a more confusing mental model for me.*

### Addresses 

The ethereum blockchcain is made up of accounts. An account has a balance of ether and can send and receive payments. Each account has a unique **address**, which looks like this:
```
0x0cE446255506E92DF41614C46F1d6df9Cc969183
```

# Lesson 1: The Zombie Factory

In this lesson, we're supposed to build a "Zombie Factory" that generates an army of zombies. 

- The factory will maintain a database of all zombies in our army
- The factory will have a function for creating new zombies
- Each zombie will have a random and unique appearance

## Zombie DNA
All zombies have DNA, which determines their appearance. It is a 16-bit integer:
```
8356281049284737
```
Like regular DNA, Zombie DNA is composed of *genes*, which correlate to specific traits. In our case, these genes are composed of 2 digits each. The first two digits map to the head type, the second two map to the color of the eyes, and so on:

![zombie dna](https://cryptozombies.io/images/feature-zombie-dna.png)

## Smart Contract Boilerplate

Solidity code is mostly built around **smart contracts**. 

> A contract is the fundamental building block of Ethereum applications — all variables and functions belong to a contract, and this will be the starting point of all your projects.

Your solidity code should start like this:
```
pragma solidity ^0.4.19;

contract HelloWorld {

}
```
## State Variables & Integers

Solidity doesn't use your average, run-of-the-mill variables. It uses... **State Variables**!

**State variables** are variables that are permanently stored in our contract's storage. This means they're written to the Ethereum blockchain :open_mouth::open_mouth:

Similarly, Solidity doesn't use just any old `int`, it uses `uint`s, or **Unsigned Integers**! This means that `uint`s are always non-negative.

> Note: In Solidity, `uint` is actually an alias for `uint256`, a 256-bit unsigned integer. You can declare `uint`s with less bits — `uint8`, `uint16`, `uint32`, etc.. But in general you want to simply use `uint` except in specific cases, which we'll talk about in later lessons.

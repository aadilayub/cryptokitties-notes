# Lesson 1: The Zombie Factory

In this lesson, we're going to build a "Zombie Factory" that generates an army of zombies. 

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

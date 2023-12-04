---
title: Misadventures in MD with VASP
date: 2023-12-03
categories: [VASP, MolecularDynamics]
tags: vasp     # TAG names should always be lowercase
---

Recently, I have been trying to simulate some carbon and boron based adsorbates to a gold surface. It seemed like a somewhat trivial task using VASP. You can run *ab initio* molecular dynamics quite simply. To demonstrate this, the following vasp `INCAR` is set to run a molecular dynamics simulation within the NVT ensemble.

#### MD NVT Input File

```bash
ISMEAR = 0
IBRION = 0
MDALGO = 2
ISIF = 2
SMASS = 1.0
SIGMA = 0.1
LREAL = Auto
#ALGO = VeryFast #for testing purposes only
#PREC = Low
ISYM = 0
TEBEG = 323
NSW = 70
POTIM = 3.0
NCORE = 2
```

You can see that there are the usual tags in there, such as __ISMEAR__, __ISIF__, __SIGMA__. They do exactly as one would expect in any regular DFT calculation with VASP. It is fun to note that __ISIF = 2__ is actually what is used to contrain the volume for the NVT ensemble. 

As for the newer tags, the self-evidently named __MDALGO__ sets the desired thermostat. In the case of __MDALGO = 2__, the [Nose-Hoover Thermostat](https://www.vasp.at/wiki/index.php/Nose-Hoover_thermostat) has been chosen to regulate the temperature of the simulation. 



### Exploding Adsorbates
___

You can see that my simulation did not go all that well. It seems like the hydrogen atoms would prefer to form hydrogen gas. This then causes the adsorbate to implode. My guess is that the forces are so strong that the atoms oscillate violently to establish an equilbrium. If my timestep (3 femtoseconds) was smaller, I'm sure we would see a smoother picture. 

![Exploding adsorbate on a gold surface][exploding adam]

My colleague suggested that the source of this strange behavior is due to improper electron counting. *How typical of a theoretical chemist with a background in inorganic chemistry.* His hypthesis is that I have a thiolate, an anionic species but I haven't accounted for the additional pair of electrons in the system. If I were to run the simulation without specifying that my adsorbate was not charge neutral, then the adsorabte or gold would have to make up for that imbalance. Consequently, two of the hydrogens would happily give up an electron to form hydrogen gas and leave behind a charge neutral adsorbate-substrate pair.

Easy enough to fix. 

```bash
ISMEAR = 0
IBRION = 0
MDALGO = 2
ISIF = 2
SMASS = 1.0
SIGMA = 0.1
LREAL = Auto
#ALGO = VeryFast #for testing purposes only
#PREC = Low
ISYM = 0
TEBEG = 323
NSW = 70
POTIM = 3.0
NCORE = 2
NELECT = 1696
```

Unfortunately, I was not gifted with the talent of *arithmetic*. You may note that I have chosen to include 1696 electrons in my system (with a starting count of 1694). This means that I have only added a single lone pair into the system, even though my simulation has two anions. 

Regardless, my simulation ran. What is quite nice is that the MD trajectory did not collapse as violently as the last time


![Less exploding adsorbate on a gold surface][less exploding adam]

The adsorbates maintain their cage structures. Unfortunately, the hydrogen atoms continue to form hydrogen gas - something I don't blame them for. 

Currently, I am running another trajectory with an additional lone pair. The only problem is that I am somewhat unsure that this is truly the solution to my problem. If you read the VASP wiki entry for [NELECT](https://www.vasp.at/wiki/index.php/NELECT), you can see the following comment

>  Warning: Unless you would like to perform a charged calculation, you should not set this line.

From my understanding, VASP assumes charge neutrality of the unit cell. This might actually remediate the charge issue and point to some other potential problem with something like my starting geometry. 

For now, I will be testing the missing electron hypothesis. 

___
#### Update:

Seems like the hydrogen atoms do not remain bound. 

![Two lone pairs][electrons]

[exploding adam]: /assets/images/adam_explode.gif
[less exploding adam]: /assets/images/adam_less_explode.gif
[electrons]: /assets/images/electrons.gif
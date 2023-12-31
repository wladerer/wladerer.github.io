---
title: vsh - A VASP cli utility 
date: 2023-12-03
categories: [Software, vsh]
tags: software, python 
---


There are many tasks that my research requires me to repeatedly - preparing input files, archiving results, checking on jobs, etc. Thankfully, my set of tools is quite small. I really only depend on VASP to run my simulations, so I realize that much of my work is well constrained. This led me to making the python based CLI known as [vsh](https://github.com/wladerer/vsh). 

Much of it is a glorified wrapper to existing codebases like pymatgen and ase. But, I have found that most computational chemistry post/pre-processing tools lack usability - especially with respect to command line functionality (phonopy, you get a pass). 

___

### A worked Example: 2D Band Structures

I want to show you just a small sample of what `vsh` is capable of by showing you a hypothetical workflow to calculate a surface projected band structure of a slab.

First, we make a directory for our slab calculation. We are going to skip the geometry relaxation because the process is nearly identical. 

```bash
mkdir slab
vsh poscar --mp-poscar 1096 -o CeS.vasp
vsh slab CeS.vasp -m 0 0 1 --freeze 5 --vacuum 10 -o CeS_001_surface 

mv CeS_001_0.vasp POSCAR
```

The penultimate command generates all possible $[001]$ terminations that are no smaller than 15 angstrom, are frozen from 5 angstrom down, and has a 10 angstrom thick vacuum. 

We can then create the requisite input files. 

```bash
vsh incar --write band -o INCAR
vsh kpoints --plane 60 -o KPOINTS
vsh potcar --potcar -o POTCAR
```

When the job finishes, you can check if the calculation has converged

```bash
vsh analysis --converged
```

And if so, you can plot a plain band structure using the following

```bash
vsh band --mode plain
```

### Design Principles
___

Ultimately, I hope to refine and prune this project so it is as close to the Unix design philosophy as possible. 


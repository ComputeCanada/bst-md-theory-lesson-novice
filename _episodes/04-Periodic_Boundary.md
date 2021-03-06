---
title: "Periodic Boundary Conditions"
teaching: 15
exercises: 0
questions:
- "How to simulate a bulk (gas, liquid or solid) system by using only a small part?"
objectives:
- "Understand why and when periodic boundary conditions are used."
- "Understand how shape and size of a periodic box can affect simulation."
- "Learn how to set periodic box parameters in GROMACS and NAMD."
keypoints:
- "Periodic boundary conditions are used to approximate an infinitely large system."
- "Periodic box should not restrict molecular motions in any way."
- "The macromolecule shape, rotation and conformational changes should be taken into account in choosing the periodic box parameters."
---
Periodic boundary conditions (PBC) are used to approximate a large system by using a small part called a unit cell. The boundary to contain molecules in simulation is needed to preserve thermodynamic properties like temperature, pressure and density. Application of PBC to simulations allows to include the influence of bulk solvent or crystalline environments.

![Figure: Periodic Boundary Conditions]({{ page.root }}/fig/periodic_boundary.png){: width="240" }

To implement PBC the unit cell is surrounded by translated copies in all directions to approximate an infinitely large system. When one molecule diffuses across the boundary of the simulation box it reappears on the opposite side. So each molecule always interacts with its neighbours even though they may be on opposite sides of the simulation box. This approach replaces the surface artifacts caused by the interaction of the isolated system with a vacuum with the PBC artifacts which are in general much less severe.

> ## Specifying Periodic Box
>  **GROMACS**
>
> The box specification is integrated into structure files. The box parameters can be set using the [*editconf*](http://manual.gromacs.org/archive/5.0/programs/gmx-editconf.html) program or manually. The *editconf* program accepts the following options:
>
>-----------------|------------------|
> *-bt* &emsp;    | Box type            | *triclinic, cubic, dodecahedron, octahedron*
> *-box* &emsp;   | Box vectors lengths, *a, b, c* | nm
> *-angles* &emsp;| Box vectors angles, *bc, ac, ab* | degrees
> *-d* &emsp;     | Distance between the solute and the box | nm
>
>Example:
>~~~
>module load StdEnv/2020 gcc gromacs
>wget http://files.rcsb.org/view/1lyz.pdb
>gmx pdb2gmx -f 1lyz.pdb -ff amber99sb-ildn -water spce -ignh
>gmx editconf -f conf.gro -o conf_boxed.gro -d 1.0 -bt cubic
>~~~
> {: .language-bash}
> In the example above the *editconf* program will append box vectors to the structure file *'conf.gro'* and save it in the file *'conf_boxed.gro'*. The 9 components of the three box vectors are saved in the last line of the structure file in the order: xx yy zz xy xz yx yz zx zy. Three of the values (xy, xz, and yz) are always zeros because they are duplicates of (yx, zx, and zy).  The values of the box vectors components are related to the unit cell vectors $$a,b,c,\alpha,\beta,\gamma$$ from the *CRYST1* record of a PDB file with the equations:
>
>$$xx=a, yy=b\cdot\sin(\gamma), zz=\frac{v}{(a*b*\sin(\gamma))}$$
>
>$$xy=0, xz=0, yx=b\cdot\cos(\gamma)$$
>
>$$yz=0, zx=c\cdot\cos(\beta), zy=\frac{c}{\sin(\gamma)}\cdot(cos(\alpha)-cos(\beta)\cdot\cos(\gamma))$$
>
>$$v=\sqrt{1-\cos^2(\alpha)-cos^2(\beta)-\cos^2(\gamma) +2.0\cdot\cos(\alpha)\cdot\cos(\beta)\cdot\cos(\gamma)}\cdot{a}\cdot{b}\cdot{c}$$
>
> **NAMD**
>
> Periodic box is specified in the run parameter file by three unit cell vectors, the units are <span>&#8491;</span>.
>~~~
> # cubic box
> cellBasisVector1 100 0 0
> cellBasisVector2 0 100 0
> cellBasisVector3 0 0 100
>~~~
>{: .file-content}
> Alternatively periodic box parameters can be read from the *.xsc* (eXtended System Configuration) file by using the *extendedSystem* keyword.  If this keyword is used *cellBasisVectors* are ignored.  NAMD always generates  *.xsc* files at runtime.
>~~~
> extendedSystem restart.xsc
>~~~
>{: .file-content}
{: .callout}

## What size/shape of a periodic box should I use?

### Box size
Solvated macromolecules rotate during simulations. Furthermore macromolecules may undergo conformational changes. Often these changes are of major interest and should not be restricted in any way. If the molecule is not spherical and the box dimension is not large enough rotation will result in the interaction between copies. This artifactual interaction may influence the motions of the system and affect the outcome of the simulation. To avoid these problems the minimum box dimension should be larger than the largest dimension of the macromolecule plus at least 10 <span>&#8491;</span>.

### Box shape
 A cubic box is the most intuitive and common choice, but it is inefficient due to irrelevant water molecules in the corners. The extra water will make your simulation run slower. Ideally you need a sufficiently large sphere of water surrounding the macromolecule, but that's impossible because spheres can't be packed to fill space. A common alternatives that are closer to spherical are the dodecahedron (any polyhedron with 12 faces) or the truncated octahedron (14 faces). These shapes work reasonably well for globular macromolecules, but if the solute is elongated there will be a large amount of the unnecessary water located far away from the solute. In this case you may consider constraining the rotational motion [[1]](https://aip.scitation.org/doi/10.1063/1.480557) and using a smaller rectangular box. But be aware that the box shape itself may influence conformational dynamics by restricting motions in certain directions [[2]](https://onlinelibrary.wiley.com/doi/full/10.1002/jcc.20341). This effect may be significant when the amount of solvent is minimal.

### Cut-off radius and box size
In simulations with PBC the non-bonded interaction cut-off radius should be smaller than half the shortest periodic box vector to prevent interaction of an atom with its image.

>## Comparing periodic boxes
>Using the structure file *'conf.gro'* from the example above generate triclinic, cubic, dodecahedral and truncated octahedral boxes with the 15 <span>&#8491;</span> distance between the solute and the box edge.
>
>Which of the boxes will be the fastest to simulate?
{: .challenge}

So what are the triclinic boxes?  There turns out that any repeating shape that fills all of space  has an equivalent triclinic box. Given a truncated octahedron, for example, there is an equivalent triclinic box with exactly the same volume that, when tiled periodically throughout space, produces exactly the same repeating pattern of atoms.  And that means that if you have triclinic boxes, you automatically have all other shapes as well.

Given the minimum required distance between periodic copies, the optimal triclinic cell has about 71% the volume of the optimal rectangular cell, which means your system only needs about 71% the number of atoms.  So although this isn't a huge optimization, it does allow you to use a somewhat smaller system that can be simulated somewhat faster.

Truncated octahedron periodic boundary conditions are isomorphic to parallelepiped boundary conditions with specific cell vectors. If you view an octahedral simulation in VMD, it will look like a parallelepiped and not a 
truncated octahedron, but they have exactly the same properties, so it doesn't matter. 

References:  
1. [Molecular dynamics simulations with constrained roto-translational motions: Theoretical basis and statistical mechanical consistency](https://aip.scitation.org/doi/10.1063/1.480557) 
2. [The effect of box shape on the dynamic properties of proteins simulated under periodic boundary conditions](https://onlinelibrary.wiley.com/doi/full/10.1002/jcc.20341)
3. [Periodic box types in Gromacs manual](https://manual.gromacs.org/current/reference-manual/algorithms/periodic-boundary-conditions.html?highlight=periodic%20boundary%20conditions)

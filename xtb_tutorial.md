---
layout: page
title: "User Guide to Semiempirical Tight Binding"
description: "An attempt to give some insights into one of my software projects"
date:   2019-03-14
permalink: /xtb-tutorial/
---

This user guide focuses on the semiempirical quantum mechanical (SQM) method
which is called extended tight binding (xTB) as it is implemented in the
`xtb` program package.

As one of the programmers behind the `xtb` program I will try to give you
some insight into `xtb` and how to use its features.
Note that, this is **not** the official manual of `xtb`
(to the best of my knowledge there is none).

{% include toc.html %}

[//]: # (Extended Tight Binding Elsewhere)
[//]: # (================================)
[//]: # ()
[//]: # (Other quantum chemistry program packages implementing xTB are)
[//]: # ()
[//]: # (* **CP2K** (GFN1-xTB))
[//]: # (* **Turbomole** (GFN0-xTB, GFN1-xTB, GFN2-xTB))
[//]: # (* **FermiONs** (GFN2-xTB))
[//]: # (* **TeraChem** (GFN1-xTB, GFN2-xTB))
[//]: # ()
[//]: # (This list might become inaccurate as time propagates.)

Getting the Program
===================

The `xtb` program is available for academic use free of charge on request
from Stefan Grimme at the [xtb-mailing list](mailto:xtb@thch.uni-bonn.de).
It usually comes as a tarball with following content

{% highlight none %}
bin/xtb
.xtbrc
Config_xtb_env.bash
Config_xtb_env.csh
man/xtb.1.html
man/xtb.1.pdf
man/man1/xtb.1
man/xcontrol.7.html
man/xcontrol.7.pdf
man/man7/xcontrol.7
info/RELEASE_NOTES.html
info/RELEASE_NOTES.pdf
{% endhighlight %}

The binary is usually compiled with the Intel® Fortran compiler and statically
linked against Intel®'s Math Kernel Library (Intel® MKL).

First check the version by

{% highlight none %}
% xtb --version
      -----------------------------------------------------------
     |                   =====================                   |
     |                           x T B                           |
     |                   =====================                   |
     |                         S. Grimme                         |
     |          Mulliken Center for Theoretical Chemistry        |
     |                    University of Bonn                     |
     |                Version 6.1 RC1 (SAW190306)                |
      -----------------------------------------------------------
{% endhighlight %}

This should print some fancy banner, the version number, say 6.1 RC1, the
last programmer worked on the project (usually `SAW`, that's me)
and the date the program was last compiled and tested by this programmer,
as `YYMMDD`. If this timestamp and the timestamp on the tarball differ
by more than a week be alert.

Setting up `xtb`
================

This section will give you the basic information you need to
know about the `xtb` program. Some of the steps are elemental
for your calculation to succeed, so please consider to follow
my instructions carefully.

Some part of the `xtb` program can be quite wasteful with stack memory,
to avoid stack overflows when calculating large molecules, you should
unlimit the system stack, *e.g.* with `bash` by

{% highlight bash %}
ulimit -s unlimited
{% endhighlight %}

Note that the memory management of `xtb` is constantly improved to avoid
using large amounts of stack memory, but to be on the save side
include this option for production runs.

Parallelisation
---------------

The `xtb` program uses OMP parallelisation, to calculate larger systems
an appropriate OMP stacksize must be provided, chose a reasonable large number by

{% highlight bash %}
export OMP_STACKSIZE=1G
{% endhighlight %}

To distribute the number of threads reasonable in the OMP section
I recommend to use

{% highlight bash %}
export OMP_NUM_THREADS=<ncores>,1
{% endhighlight %}

You might want to deactivate nested OMP constructs by

{% highlight bash %}
export OMP_MAX_ACTIVE_LEVELS=1
{% endhighlight %}

Environment Variables for `xtb`
-------------------------------

A number of environment variables is used by `xtb` to perform calculations.
Please set the `XTBPATH` variable to include all locations were
you store information relevant for your `xtb` calculation, like configuration
files and parameter files.
The present working directory is implicitly included for most files that
are searched in the `XTBPATH`.

The old `XTBHOME` variable is used if you have not set the `XTBPATH`
variable and is used in the same manner. `xtb` will print the values
of `XTBPATH` and `XTBHOME` at the beginning of each calculation
if set to verbose mode.

Getting Help
============

Beside this manual you are almost on your own. You can check the
in-program help by

{% highlight bash %}
% xtb --help
{% endhighlight %}

Unfortunately, this might be outdated, since nobody feels responsible for it,
therefore, you should refer to the man-pages distributed with the `xtb` program.
Please check for the man-pages of `xtb(1)` and `xcontrol(7)`.

The Verbose Mode
----------------

If you think some information is missing in your calculation you can
switch to the verbose mode by using `--verbose` in the command line
arguments. This will increase the print level almost everywhere in the
`xtb` program, also the input parser will print a lot of information
that might be interesting for your current calculation.

Overall this can be an awful lot of information, so it is not recommended
as a default option.

Control by Command-line
======================

The program can almost entirely controlled by the command-line, if you
need more control you should resort to the detailed input file.
There are four main run types in `xtb`, most other run types are
composite types that try to provide convenient combinations from
those main run types.

Singlepoint Calculations
------------------------

Independent of all other commands, there will always be a singlepoint
calculation carried out at the very beginning. To calculate something
`xtb` needs information about the geometry of the atoms

The default input format is either the Turbomole coordinate file
as a `$coord` data group starting in the very first line:

{% highlight none %}
% cat coord
$coord
    0.00000000000000      0.00000000000000     -0.73578586109551      o
    1.44183152868459      0.00000000000000      0.36789293054775      h
   -1.44183152868459      0.00000000000000      0.36789293054775      h
$end
% xtb coord
{% endhighlight %}

Any valid Xmol file (`xtb` will actually count the lines and double check
the number of atoms specified), here the suffix xyz is optional since `xtb`
will auto detect the file type.
`xtb` also supports structure-data files (sdf), if the corresponding suffix
is encountered.

By default `xtb` will search for `.CHRG` and `.UHF` files and obtain
from these the molecular charge and the number of unpaired electrons,
respectively. The molecular charge can also be specified by

{% highlight bash %}
% xtb molecule.xyz --chrg +1
{% endhighlight %}

which is equivalent to

{% highlight bash %}
% echo +1 > .CHRG && xtb molecule.xyz
{% endhighlight %}

This also works for the unpaired electrons as in

{% highlight bash %}
% xtb --uhf 2 input.sdf
{% endhighlight %}

Note that the position of the input coordinates is totally unaffected
by any command-line arguments, if you are not sure, whether `xtb` tries
to interpret your filename as flag use `--` to stop the parsing
as command-line options for all following arguments.

{% highlight none %}
% xtb -- -oh.xyz
{% endhighlight %}

To select the parametrization of the xTB method you can currently choose
from three different geometry, frequency and non-covalent interactions (GFN)
parametrization, which differ mostly in the cost--accuracy ratio.

{% highlight bash %}
% xtb --gfn 2 coord
{% endhighlight %}

to choose GFN2-xTB, which is also the default parametrization. Also
available are GFN1-xTB and GFN0-xTB.

Sometimes you might face difficulties converging the self consistent
charge iterations, in this case it is usually a good idea to increase
the electronic temperature and to restart at normal temperature

{% highlight bash %}
% xtb --etemp 1000.0 coord && xtb --restart coord
{% endhighlight %}

Geometry Optimizations
----------------------

The main purpose of the xTB methods is to provide good geometries,
so the `xtb` comes with a build-in geometry optimizer, which usually
does a decent job. It is invoked by

{% highlight bash %}
% xtb coord --opt
% ls
coord   xtbopt.coord   xtbopt.log   ...
{% endhighlight %}

The optimized coordinates is written to a new file (`xtbopt.coord`), which is
in the same format as the input geometry. You can view the geometry optimization
by opening the `xtbopt.log` with your favorite molecule viewer.
The log-file is in Xmol format and contains the current total energy
and the gradient norm in the comment line, `gmolden` usually works fine
for this.

A successful geometry optimization will print
`*** GEOMETRY OPTIMIZATION CONVERGED AFTER 39 ITERATIONS ***`
after finishing the optimization procedures, while in all other cases
that not exit in error
`*** FAILED TO CONVERGE GEOMETRY OPTIMIZATION IN 500 ITERATIONS ***`
will be printed, additionally a `NOT_CONVERGED` file is created in the
working directory, which might become handy for bulk jobs.

To get a geometry optimization to converge can be a hard job, usually
the xTB methods can repair a lot, you might want to start from GFN0-xTB
which does not have convergence issues and than improve with GFN2-xTB.
Maybe you have to adjust the geometry by hand again, if even this fails.

`xtb` offers eight predefined levels for the geometry optimization,
which can be chosen appending the level to the optimization flag as

{% highlight bash %}
% xtb coord --opt tight
{% endhighlight %}

The thresholds defined by simple keywords are given here

|   level | Econv/Eh | Gconv/Eh·α⁻¹ | Accuracy |
|:--------|---------:|-------------:|---------:|
| crude   | 5 × 10⁻⁴ | 1 × 10⁻²     | 3.00     |
| sloppy  | 1 × 10⁻⁴ | 6 × 10⁻³     | 3.00     |
| loose   | 5 × 10⁻⁵ | 4 × 10⁻³     | 2.00     |
| lax     | 2 × 10⁻⁵ | 2 × 10⁻³     | 2.00     |
| normal  | 5 × 10⁻⁶ | 1 × 10⁻³     | 1.00     |
| tight   | 1 × 10⁻⁶ | 8 × 10⁻⁴     | 0.20     |
| vtight  | 1 × 10⁻⁷ | 2 × 10⁻⁴     | 0.05     |
| extreme | 5 × 10⁻⁸ | 5 × 10⁻⁵     | 0.01     |

The energy convergence (Econv) is the allowed change in the total energy
at convergence, while the gradient convergence (Gconv) is the 
allowed change in the gradient norm at convergence. The accuracy
is handed to the singlepoint calculations for integral cutoffs and
self consistent field convergence criteria and is adjusted to fit
the geometry convergence thresholds automatically.

The xTB methods are completely analytical, so you can in principle
converge your results down to machine precision. Converging it
down to the lower limit is more a development feature than a
real life application but always possible.

Characterisation of Stationary Points
-------------------------------------

In `xtb` second derivatives are implemented by finite differences methods 
(numerical second derivatives). Normally you want to calculate the Hessian
directly after a successful geometry optimization, this is done by using

{% highlight bash %}
% xtb coord --ohess
{% endhighlight %}

For the calculation on the input geometry use `--hess` instead.

### Dealing with Small Imaginary Frequencies

For small imaginary modes `xtb` offers an automatic distortion feature
of these modes, say you have optimized a geometry and performed
a frequency calculation which leads to an imaginary frequency of
14 wavenumbers:

{% highlight none %}
% xtb coord --ohess
 ...
           -------------------------------------------------
          |               Frequency Printout                |
           -------------------------------------------------
 projected vibrational frequencies (cm-1)
eigval :       -0.00    -0.00     0.00     0.00     0.00     0.00
eigval :      -14.26     8.12     9.26    12.09    15.85    17.73
eigval :       19.45    28.85    39.18    41.30    64.61    71.84
 ...
imag cut-off (cm-1) :    5.00
 found            1  significant imaginary frequency
 writing imag mode distorted coords to <xtbhess.coord>
 for further optimization.
 ...
{% endhighlight %}

In this case `xtb` will generate a distorted structure, you can continue to
optimize with

{% highlight none %}
% xtb xtbhess.coord --ohess
 ...
           -------------------------------------------------
          |               Frequency Printout                |
           -------------------------------------------------
 projected vibrational frequencies (cm-1)
eigval :       -0.00    -0.00    -0.00    -0.00     0.00     0.00
eigval :        2.02     7.99    10.10    12.08    16.16    18.57
eigval :       23.88    28.93    38.35    42.18    64.86    73.76
 ...
{% endhighlight %}

The optimization will only take a few steps and the artifical imaginary
frequency is gone after checking frequency calculation.

The Detailed Input: `xcontrol`
==============================

The `xcontrol` instruction set is inspired by the Turbomole `control`
file syntax. I decided to call it `xcontrol` instructions back than,
but here we will just call it (detailed) input for convenience.
As a sitenote: The parser is actually not limited to this choice
of syntax and could be without much effort extended to something more
general like JSON or YAML.
To read an input file called `xtb.inp` use

{% highlight bash %}
% xtb --input xtb.inp coord
{% endhighlight %}


In the detailed input you have control about almost very global
variable in the program, some instructions even check your input, but
most of the time you should know what you are doing.
Developed as a feature for developers, this is incredible powerful
and naturally way to complicated for the average application.
So in most cases you can safely rely on the internal defaults or
the shipped global configuration file (should usually be the same).

I will walk you through some selected instructions you might find useful
for your application.

Fixing, Constraining and Confining
----------------------------------

In `xtb` different concepts of constraints are implemented,
so you should know which tool is best for you problem before you
start writing the detailed input.

### Exact Fixing

In the *exact fixing* approach the Cartesian position of the selected
atom is fixed in space by setting its gradient to zero and the degrees
of freedom are removed from the optimization procedure and therefore
the atoms stay in place in geometry optimizations.

For dynamics this exact fixing is *automatically deactivated*, since it
usually leads to instabilities in the simulation.

To activate the exact fixing for atoms 1--10 and atom 12 as well as for
all oxygen atoms, add

{% highlight bash %}
$fix
   atoms: 1-10,12
   elements: O
$end
{% endhighlight %}

to your detailed input, the atoms keyword refers to the numbering
of the individual atoms in your input geometry.

### Constraining Potentials

Almost absolute control about anything in your system is archived
by applying *constraining potentials*. First of all the constraining
potentials offer a weaker version of the exact fixing, which is
invoked by the same syntax in the `$contrain` data group as

{% highlight bash %}
$constrain
   atoms: 11
   elements: C,N,8
$end
{% endhighlight %}

the program will not attempt to hold the Cartesian positions constant,
but the distances between all selected atoms, here number 11 and all
carbon, nitrogen and oxygen. For each atom pair a harmonic potential
is generated to hold the distances at roughly the starting value, this even
works without problems in dynamics.

To constrain the atoms more tightly the force constant can be adjusted

{% highlight bash %}
$constrain
   force constant=1.0
$end
{% endhighlight %}

this variable goes directly into the constraining procedure and is given in
Hartree, for very high force constants this becomes equivalent to the exact fixing.
Note the difference in the syntax as you are required to use an equal-sign
instead of a colon, as you are modifying a global variable.

It is also possible to constrain selected internal coordinates, possible
are distances, angles and dihedral angles as done here

{% highlight bash %}
$constrain
   distance: 1, 2, 2.5
   angle: 5, 7, 8, 120
   dihedral: 3, 4, 1, 7, auto
$end
{% endhighlight %}

Distance constraints are given in Ångström, while angle constraints are given
in degrees.
The distances are defined by two atom number referring to the order in
your coordinate input, angles are defined by three atom numbers and
dihedral angles by four atoms, in any case the atoms do not have to
be connected by bonds. The last argument is always the value which should
be used in the constraining potential as reference, if you decide to
use the current value `auto` can be passed. The constraints will be
printed to the screen (the newer implementation may require the verbose mode,
to trigger the printout of the constraint summary).

If you are not quite sure which distances or angles you want to constrain,
run

{% highlight bash %}
% cat geosum.inp
$write
   distances=true
   angles=true
   torsions=true
$end
% xtb --define --verbose --input geosum.inp coord
{% endhighlight %}

and have a look at the geometry summary for your molecule. The `$write`
data group toggles the printout in the property section and also some
printouts in the input section.

### Confining in a Cavity

If you are running dynamics for systems that are non-covalently bound,
you may encounter dissociation in the dynamics. If you want to
study the bound complex, you can try to *confine* the simulation
in a little sphere, which keeps the molecules from escaping.
The detailed input looks like

{% highlight bash %}
$wall
   potential=logfermi
   sphere: auto, all
$end
{% endhighlight %}

You can be more precise on the radius by giving the value in bohr instead
of `auto`. I personally recommend to use the logfermi potential, since it
is best suited for confinements, but yet not the default.

References
==========

*  C. Bannwarth, S. Ehlert and S. Grimme,
   *J. Chem. Theory Comput.*, **2019**, 15 (3), 1652--1671.
   [DOI: 10.1021/acs.jctc.8b01176](https://doi.org/10.1021/acs.jctc.8b01176)

*  S. Grimme, C. Bannwarth and P. Shushkov,
   *J. Chem. Theory Comput.*, **2017**, 13, 1989--2009.



# Replication codes: "Aggregate effects of firing costs with endogenous firm productivity growth"

This repository includes the replication package for my paper "[Aggregate effects of firing costs with endogenous firm productivity growth](https://borjapetit.github.io/files/papers/petit_firing.pdf)". The codes are writen in Fortran and makes use of `OpenMP` for parallelization and `GNUplot` to generate the graphs.  `OpenMP` is included in most Fortran distributions, and `GNUplot` can be freely downloaded [here](https://sourceforge.net/projects/gnuplot/files/gnuplot/5.2.7/). If you don't want to download it, you will need to load the data from a number of `.txt` files (located in the folder `~/results`) into Excel, Matlab, etc, to generate the figures.

Please, if you find any error or have a suggestion, do not hesitate to  [contact](mailto:bpetit@cunef.edu) me.

## Content

| File/folder | Description |
|:------------|:-------------|
| `main.f90` | This file governs the solution of the model. |
| `calibration.f90` | This file contains the functions and subroutines required to calibrate the parameters of the model. |
| `experiment.f90` | This file contains the subroutines that implement the experiments and the sensitivity analysis. |
| `firmsproblem.f90` | This file contains the subroutines and function that solve the firm's problem, compute the invariant distribution of firms, and calculate the moments used to calibrate the model. |
| `params.f90` | This file contains all the global variables used throughtout the paper, as well as a number of simple functions and subroutines to print results and generate graphs. |
| `toolkit.f90` | This file contains a number of general-purpose functions and subroutines such as minimization algorithms, algebra, etc. |
| `~/compiledfiles` |  This folder stores the figures generated by the program, and contains a number of `.gnu` files with instructions to generate graphs in `GNUplot` located in the subfolder `~/figures/code` |
| `~/results` | This folder stores `.txt` files with different results from the model |
| `~/results/graphs` | This folder stores the `.txt` files with data that are then used to generate graphs |
| `~/txtfiles` | This folder contains two `.txt` files: (i) `params.txt` which contains the parameter values, and  (ii) `moments.txt`containing the empirical moments used for calibration. |


**IMPORTANT**: It is important that you keep the `.f90` files and the four folders in the same directory. 

## Compilation

To compile the codes, you first need to set the current directory:
```
cd /Users/(...)/code/
```
and then use the following command:
```
gfortran -fopenmp -O3 -ffixed-line-length-132 -J $(pwd)/compiledfiles toolkit.f90 params.f90 firmsproblem.f90 calibration.f90 experiment.f90 main.f90 -o model
```
This command includes four flags. First, compiler to link the executable with `OpenMP` for parallel programing, using the flag `-fopenmp`. Second, the flag `-O3`, makes the code faster, although it slows down compilation a bit. Third, the flag `-ffixed-line-length-132` prevents line truncation (I set the line length to 132). Finally, and in order to keep the working directory clean, the flag `$(pwd)/compiledfiles` forces the compiler to store the `.mod` files in the folder `~/compiledfiles` (this will only work is you have defined your current directory before).

If you are using the Intel compiler, the command is:
```
ifort /openmp /O3 /object:%cd%\compiledfiles\ /module:%cd%\compiledfiles\ toolkit.f90 params.f90 firmsproblem.f90 calibration.f90 experiment.f90 main.f90 -o model
```

**IMPORTANT**: Before compiling the code the user needs to specify the working directory in the variable `path`, in `params.f90`, since all the I/O files are written relative to the specified path.
  
## Solution

Once compiled, you can run the code by double-clicking the generated executable. The code can be used to solve the model, to run the experiments, to calibrate the parameters, and to run a robustness check of the model parameters.

When clicking the executable, the program will aks you to specify what to do:

```
What do you want to do?
  [ 0 ] Solve the model for given parameters (params.txt)
        -> take value of theta from params.txt & find wage
  [ 1 ] Solve the model for given parameters (params.txt)
        -> assume wage rate = 1, and compute the theta
  [ 2 ] Run experiment
  [ 3 ] Calibrate the parameters
  [ 4 ] Run sensitivity analysis
```

The difference between `[0]` and `[1]` is that the former takes the value of the labor disutility parameter in the household's problem (`theta`) from the text file `params.txt` and compute the equilibrium wage. If you choose `[1]`, instead, the program will asumme a wage rate of one and compute the labor disutility parameter that makes the household's first order condition to be satisfied.

If you choose `[0]` or `[1]`, and after the solution is computed, the program will ask you whether you want to produce figures.

```
  Do you want to generate graphs? [1] Yes, [0] No
```

The generated graphs will be stored in the folder `~/figures` and will be named `baseline_(...).pdf`. The data used to generated those graphs will be stored in `~/results` and will be named `baseline_(...).txt`. You can check which graphs are produced in the subroutine `PRINTGRAPHS`, in `params.f90`.


## Experiments

If you choose to **run the experiment**, typing `[2]`, you will be asked which experiment you want to run. There are 3 options:
```
Which experiment do you want to run
  [ 1 ] Varying firing costs: GE & PE
  [ 2 ] Varying firing costs: with & w/o innovation 
  [ 3 ] Varying firing costs: with innovation & static AR(1)   
  [ 4 ] Varying firing costs: with innovation & changing AR(1) 
  [ n ] n points in (0,...,fc_0,...,2*fc_0), min 5
```
**Experiment 1** reports the % fall relative to the frictionless economy in a number of aggregates for an economy with the baseline value for the firing cost (`fc_0`), for an economy with a firing cost parameters twice as large as in the baseline calibration, and for an economy with a firing cost of 1. The results are reported for both the general equilibrium case (in which the wage rate is recomputed for each counterfactual economy) and the partial equilibrium one (in which the wage rate is fixed at its value in the frictionless economy). The results from this experiment are printed to `results/experiment_1.txt`.

**Experiment 2** replicates the same exercise of **Experiment 1**, but instead of solving the partial equilibrium results, it computes the case in which the innovation choices of firms are kept fixed at their values in the frictionless economy for every counterfactual economy. The results from this experiment are printed to `results/experiment_2.txt`.

**Experiment 3** replicates the same exercise of **Experiment 1**, but it computes the losses from firing costs in a economy with a (contat) AR(1) producitivity process. When solving the frictionless economy with no firing frictions, the program stores the AR(1) that better fits firms' productivity dynamics and then uses this AR(1) process for an economy with firing costs equal to zero, the calibrated alue, twice the calibrated value and 1. The results from this experiment are printed to `results/experiment_3.txt`.

**Experiment 4** replicates the same exercise of **Experiment 3**, but using the AR(1) that better fits the productivity dynamics in the baeline economy with each value of the firing costs. Thus, this experiment is similar to **Experiment 3** but changing the AR(1) process for each value of the firing cost parameter. The results from this experiment are printed to `results/experiment_4.txt`.

In both **Experiment 1** and **Experiment 2** the program calls <c>GNUplot</c> to generate some graphs illustrating the differences in average growth rates and innovation volatility for each value of the firing cost parameter.

**Experiment `n`** constructs a grid of points for the firing cost parameters ranging from 0 to twice the baseline value and solves three economies (general equilibrium, partial equilibrium, and general equilibrium with innovation choices from frictionless economy) for value of the firing cost parameter. After computing these results, the program will call `GNUplot` to generate the graphs included in appendix of the paper. If you do not have `GNUplot` installed, the program will return an error, but the results will be saved in a number of text files so you can use other softwares to generate the graphs.


## Calibration

If you choose to **calibrate** the parameters of the model, typing `[3]`, you will be asked to choose the calibration method. There are 5 options:
```
Choose calibratrion algorithm
  [ 1 ] Levenberg-Marquardt
  [ 2 ] Simplex
  [ 3 ] Random search
  [ 4 ] Levenberg-Marquardt, shocking initial values
  [ 5 ] Simplex, shocking initial values
```

The [**Levenberg-Marquardt**](https://en.wikipedia.org/wiki/Levenberg–Marquardt_algorithm) algorithm is a Newton-based method to minimize the sum of squared errors of a system of _m_ equations in _n_ unknowns. The [**Simplex**](https://en.wikipedia.org/wiki/Simplex_algorithm) method, a.k.a. Nelder-Mead method, is one of the most popular minimization algorithms to minimize a single-valued equation in _n_ unknowns. If you chose any of these, the program will ask you to set the maximum number of iterations.

The **Random search** algorithm is a simple iterative process to minimize the sum of squared errors of a system of _m_ equations in _n_ unknowns. Starting from an initial set of parameters, the algorithm shocks the parameters until a better set of parameter is found, and the algorithm restart from that set of parameters. If you choose this method, the program will ask you to set the maximum hours you want the algorithm to run.

The last two options take an initial set of parameters and minimize the sum of squared errors applying either the [Levenberg-Marquardt](https://en.wikipedia.org/wiki/Levenberg–Marquardt_algorithm) or the [Simplex](https://en.wikipedia.org/wiki/Simplex_algorithm) method. Then, the program shocks the initial parameter values one by one, and keeps iterating over the initial set until convergence.

## Robustness

The last thing you can do with these codes is to run a simple robustness check of the main results, as included in the appendix of the paper. In particular, the program recomputes the general equilibrium results included in "Experiment 1" first for the bechmark calibration, and then shocking each parameter at a time.

The results from this exercises are collected in `results/sensitivity.txt`.

   



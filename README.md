# RANSA (Receptor Array Nested Sampling Algorithm) User Guide

* [Overview][0]
* [Compilation][1]
* [Options][2]
* [To run code for Calibration or Inference][3]
* [To run code for Optimization of parameters in sensor array design][4]
* [Model Files][5]
* [Prior Files][6]
* [Examples][7]

## [Overview][8]
Copyright: © 2011 Julia Tsitron and Alexandre V. Morozov

This document provides directions for using the RANSA (C++) package.  For complete details about the design and applications of the algorithm please refer to the following two papers:

Tsitron J, Ault AD, Broach JR, Morozov AV (2011) [Decoding Complex Chemical Mixtures with a Physical Model of a Sensor Array.](http://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1002224) **PLoS Comput Biol** 7(10): e1002224\. doi:10.1371/journal.pcbi.1002224 

_and_

Tsitron J, Kreller CR, Sekhar PK, Mukundan R, Garzon FH, Brosha EL, Morozov AV. (2014) [Bayesian Decoding of
the Ammonia Response of a Zirconia-based Mixed-Potential Sensor in the Presence of Hydrocarbon Interference.](http://www.sciencedirect.com/science/article/pii/S0925400513013178)
Sens. Actuators B: Chem. 192: 283-93.

## [Compilation][8]

Requirements:

* `libgsl` ([http://www.gnu.org/software/gsl/][10])

To compile the executable \`sample', run:

**`make`**

_or_

**`make sample`**

## [Options][8]

****`-f` filename****
> 
> Load Data file

****`-model` model\_type****
> 
> Choose class of models
> 
> DEFAULT: **`recX_ligY`**
> 
> All Possible modes:
> 
> * Calibration and Inference (GPCR paper):
> 
> **`recX_ligY`**
> 
> **`MULT_SIG_recX_ligY`**
> 
> **`MuTOT_recX_ligY`**
> 
> (**`X`** = number of receptors, **`Y`** = number of ligands. Both are _single_ digits)
> * Optimization of array design:
> 
> **`FULL_HESS_recXX_ligYY`**
> 
> (**`XX`** = number of receptors, **`YY`** = number of ligands. Both are _double_ digits)
> * Calibration and Inference (gas sensors paper):
> 
> **`LINEAR_Xrec_Ylig`**
> 
> **`NONLIN_CALIB_Xrec`**
> 
> **`NONLIN_PRED_Xrec`**
> 
> (**`X`** = number of receptors, **`Y`** = number of ligands. Both are _single_ digits)

****`-gapfile` gapfilename****
> 
> Load concentration \`gap' file with dilution steps.
> 
> (Not needed in **`recX_ligY`** and **`FULL_HESS_recXX_ligYY`** modes)

****`-mfile` modelfilename****
> 
> Load file with fixed model parameters

****`-pfile` priorfilename****
> 
> Load file with priors for each parameter
> 
> DEFAULT:
> 
> * ΔG: Uniform \[-20 +5\] (All modes in GPCR paper)
> * A, b: Uniform \[0 1\] (All modes in GPCR paper)
> * V0: Uniform \[-1 1\] (All modes in gas sensors paper)
> * A (slope): Uniform \[0 1\] (All modes in gas sensors paper)
> * σ: Jeffreys \[0.0001 100.0\] (ALL modes)
> * α: Jeffreys \[0.0001 100.0\] (ALL modes)
> * μtot: Uniform\[-10 -2\] (All modes in GPCR paper)
> * \[TOTAL\]: Uniform\[0.0001 1000.0\] (All modes in gas sensors paper)

****`-n` number of nested objects****
> 
> DEFAULT: 100

****`-max` number of nesting iterations****
> 
> DEFAULT: 1000

****`-mc` number of MCMC trials for finding a new object****
> 
> DEFAULT: 20

****`-D` size of unweighted posterior sample****
> 
> DEFAULT: 1000

****`-out` posterior\_filename****
> 
> File with _unweighted_ posterior samples
> 
> (nested sampling results in a posterior distribution where each sample is weighted by its log likelihood. Here, you can output an _unweighted_ sample generated by MCMC)

****`-rs` random seed****
> 
> change random seed from default value

## [To run code for Calibration or Inference][8]

For analysis of data featured in [GPCR-based sensor array paper][9],

Use one of the following Options:

**`-model` `recX_ligY`**

**`-model` `MULT_SIG_recX_ligY`**

**`-model` `MuTOT_recX_ligY`**

You will need a Data file consisting of three columns:
    
       mu        I      recN
       -3       .99       1
     -3.5       .92       1
       -4       .84       1
       -3       .98       2 
     -3.5       .81       2
       -4       .80       2
      ...       ...      ...    
      ...       ...      ...
      ...       ...      ...    
    

(where `mu` = log₁₀\[total conc\], `I` = Intensity, and `recN` = receptor\_number)

**`recX_ligY`** mode:

* fit for any or all of the following: ΔG's, A's, b's, α's, and σ
* usually used for **Calibration** step
* there is a single σ (noise parameter) for _ALL_ of the data that you input

**`MULT_SIG_recX_ligY`** mode:

* fit for any or all of the following: ΔG's, A's, b's, α's, and multiple σ's
* there are **`X`** σ's, one for each receptor (i.e., the noise parameters associated with each receptor's data set need not be the same.)

**`MuTOT_recX_ligY`** mode:

* fit for any or all of the following: ΔG's, A's, b's, α's, σ's, and μtot (**μtot is defined as log₁₀\[total conc\] at the reference point** - see METHODS in the paper)
* usually used for **Inference** step
* in this mode, the **first column of the Data file is ignored**. Instead of reading the total concentration from the Data file, you can fit for it at a reference point, and provide a separate \`gap' file that will input the dilution steps
* so, in addition to inputting a Data file, **you will need a concentration \`gap' file** which includes the dilution steps. A dilution step is defined as the difference of logs (base 10) of the total concentrations of two consecutive measurements. The format of the gapfile is a single column:

        deltaValue
        0.0
        0.5
        1.0
        1.5
        2.0
        2.5
        3.0
        3.5
        6.0
    

in this example, the gapfile could correspond to the first column in the following Data file:
    
      mu           I        recN
      -3           1          1
    -3.5        0.98          1
      -4        0.99          1
    -4.5        0.77          1
      -5        0.44          1
    -5.5        0.14          1
      -6        0.04          1
    -6.5        0.02          1
      -9        0.02          1
    

here, log₁₀\[total conc\] at the reference point is -3\.

For analysis of data featured in [gas-sensors paper][0],

Use one of the following Options:

**`-model` `LINEAR_Xrec_Ylig`**

**`-model` `NONLIN_CALIB_Xrec`**

**`-model` `NONLIN_PRED_Xrec`**

You will need a Data file consisting of three columns:
    
       TOT       V      recN
        0       .0006         1
        0       .0006         1
        0       .0005         1
       20       .0009         1 
       20       .0009         1
       20       .0009         1
      ...       ...           1
      ...       ...           1 
        0       .0006         2
        0       .0006         2
        0       .0006         2
       20       .0010         2 
       20       .0011         2
       20       .0010         2
      ...       ...           2
      ...       ...           2
      ...       ...          ...    
      ...       ...          ...
      ...       ...          ...    
    

as well as a corresponding concentration gap file (same as described [above][11]). Here, `TOT` = \[TOTAL\], `V` = Voltage Response, and `recN` = receptor\_number). In this example, there are 3 measurements at each concentration.

**`LINEAR_Xrec_Ylig`** mode:

* fit for any or all of the following: V0's, A's, α, σ's and TOTAL
* `LINEAR_1rec_1lig` usually used for **Calibration** step with α, TOTAL fixed
* `LINEAR_Xrec_Ylig`, `Y` not equal to 1, usually used for **Prediction** step

**`NONLIN_CALIB_Xrec`** mode:

* fit for: C's, p's, σ's, while keeping α and TOTAL fixed
* Number of ligands is fixed to 2

**`NONLIN_PRED_Xrec`** mode:

* fit for α and TOTAL
* must provide a [Model File][5] with fixed parameters a, b, a', b', c'

## [To run code for Optimization of parameters in sensor array design][8]

Use the Option

**`-model` `FULL_HESS_recXX_ligYY`**

* It is not necessary to input a Data file in this mode.
* fit for any or all of the following: ΔG's, A's, b's, α's, and μtot

## [Model Files][8]

Any or all of the parameters can be fixed in any mode by including a model file.

For all modes in GPCR paper, use the following format:
    
      Prm         Val       OnOff       Ind
      ddG          -5         0          1
        A           1         0          1
        B           0         1          1
    
      ddG          -5         0          2
        A           1         0          2
        B           0         1          2
    
    alpha           1         1          1
    sigma       .0001         1          1
    mutot          -3         1
    

You can fix any of the following `Prm`s:
    
    ddG
    A
    B
    alpha
    sigma
    mutot
    

and set them to any `Val` (keeping in mind the priors you are using). `OnOff` can be either `0` or `1`. `1` will fix the given `Prm` at the indicated `Val` whereas `0` will allow that parameter to be fit freely. Setting the `OnOff` switch to `0` is equivalent to excluding that parameter from the model file.

`Ind` is the index label of a given parameter. Since there are (**`X`** × **`Y`** `= N`) `ddG`'s, `A`'s, and `B`'s the `Ind` of these parameters will run from `1` to `N`. There are (**`Y`**`-1`) `alpha`'s, **`X`** `sigma`'s and only one `mutot` - so **no** `Ind` **label is necessary for** `mutot`.

The order of the index labels for `ddG`, `A`, and `B` follows this pattern:rec\# lig\#Ind

rec1 lig1`1`

rec1 lig2`2`

rec1 lig3`3`

rec2 lig1`4`

rec2 lig2`5`

rec2 lig3`6`

rec3 lig1`7`

......

The order of the index labels for `alpha` follows this pattern:alphaInd

`alpha1` = lig2/lig1`1`

`alpha2` = lig3/lig1`2`

`alpha3` = lig4/lig1`3`

......

The index label for `sigma` is just the receptor label.

An example of a Model File for `LINEAR_Xrec_Ylig` mode from gas sensors paper:
    
      Prm         Val       OnOff       Ind
      V_0        -.0032       1          1
        A        .00089       1          1
    
      V_0       .00006        1          2
        A       .00058        1          2
    
    alpha           1         0          1
    sigma       .0001         0          1
    TOTAL         200         0
    

Where you can fix any of the `Prm`s:
    
    V_0
    A
    alpha
    sigma
    TOTAL
    

## [Prior Files][8]

Any or all of the priors can be changed by including a prior file in the following format:
    
      Prm    Val1    Val2   PriorType   Ind 
      ddG   -20.0     0.0       u        1 
        A     0.0     2.0       u        1 
        B     0.0     1.0       u        1 
    
      ddG   -20.0     0.0       u        2 
        A     0.0     2.0       u        2 
        B     0.0     1.0       u        2 
    
    alpha    0.01   100.0       j        1 
    sigma   0.001   100.0       j        1
    

You can modify the `PriorType` and/or the _lower_ and _upper_ bounds (`Val1` and `Val2`, respectively) of the prior for any `Prm`. For the Gaussian prior, `Val1` and `Val2` are the _mean_ and _standard deviation_, respectively.

The possible `PriorType`s are:

`u` (Uniform)

`j` (Jeffreys)

`g` (Gaussian)

The `Ind` labels follow the same pattern as described above (see the section on [Model Files][5])

## [Examples][8]

**1\. Calibration**

To see an example of a Calibration run, you will need to load a Data file (or use the one provided in the `~/RANSA/examples/` directory called `UDPglc_H-20.dat`):

**`./sample -f UDPglc_H-20.dat`**

The end of the output should look like:
    
    dG1 = -6.2547 +- 0.215176
    A1 = 0.750912 +- 0.0555299
    B1 = 0.0299895 +- 0.0262493
    sigma1 = 0.0517271 +- 0.0235276
    log(sigma1) = -3.08013 +- 0.504612
    

To see how parameter estimation improves with an increased number of nested iterations, try:

**`./sample -f UDPglc_H-20.dat -max 15000`**

Now, the output should look like:
    
    dG1 = -6.29776 +- 0.0580941
    A1 = 0.746728 +- 0.0177048
    B1 = 0.0128173 +- 0.00841609
    sigma1 = 0.0364009 +- 0.00488333
    log(sigma1) = -3.32185 +- 0.130941
    

If you want to plot a histogram of your results, you will need an unweighted posterior distribution for each parameter. You can output these results with, say, 10,000 samples into a file called Posterior\_sample.out. Also, try running with a different random seed to see if your previous results are stable:

**`./sample -f UDPglc_H-20.dat -max 15000 -out Posterior_sample.out -D 10000 -rs 54345`**

**2\. Inference**

To see an example of an Inference run, load a Data file and a gapfile (you can use the ones provided in the `~/RANSA/examples/` directory called `UDPgal-UDPglc_4rec.dat` and `deltaFile_4rec.dat`) as well as a Model file with all of the ΔG's, A's, b's, and σ's fixed (you can use the one provided called `model_4rec_4lig.dat`). The objective here is to fit for α's and μtot.

Since we are inputting parameters for four receptors and four ligands in `model_4rec_4lig.dat`, we need to use the `MuTOT_rec4_lig4` mode. `UDPgal-UDPglc_4rec.dat` is the data collected from four experiments, each using a mixture of lig1 (UDPGal) and lig2 (UDPglc) so we expect that `alpha1` = lig2/lig1 should be ≅ 1, and `alpha2` = lig3/lig1 and `alpha3` = lig4/lig1 should both be ≅ 0\. `mutot` should be ≅ -3\. So, if you run:

**`./sample -f UDPgal-UDPglc_4rec.dat -model MuTOT_rec4_lig4 -gapfile deltaFile_4rec.dat -mfile model_4rec_4lig.dat -max 15000`**

Your output should include the following lines:
    
    alpha1 = 1.00997 +- 0.0867217
    
    alpha2 = 0.00277454 +- 0.00643838
    
    alpha3 = 0.0114507 +- 0.00107997
    
    mutot = -2.9861 +- 0.0159282
    

**3\. Optimization of parameters in sensor array design**

For a simple example of an Optimization run, try optimizing array parameters for a one-receptor, two-ligand system. You can use the Model file provided in the `~/RANSA/examples/` directory called `model_FULL_HESS_example.dat` which fixes the `B`'s to 0, `mutot` to -3, and `alpha` to 0.25\. The ΔG's and A's are free to be optimized. Run:

**`./sample -model FULL_HESS_rec01_lig02 -mfile ~/RANSA/examples/model_FULL_HESS_example.dat -max 20000`**

Your output should include the following lines:
    
    dG1 = -11.4413 +- 0
    A1 = 0.999991 +- 0
    
    dG2 = -11.8439 +- 0
    A2 = 0.000124153 +- 0
    

which indicates that this system is optimized when one ligand is a strong agonist (`A1` ≅ 1) and the other is a strong antagonist (`A2` ≅ 0).


[0]: #overview
[1]: #compilation
[2]: #options
[3]: #to-run-code-for-calibration-or-inference
[4]: #to-run-code-for-optimization-of-parameters-in-sensor-array-design
[5]: #model-files
[6]: #prior-files
[7]: #examples
[8]: #TOC
[9]: http://www.ploscompbiol.org/article/info%3Adoi%2F10.1371%2Fjournal.pcbi.1002224
[10]: http://www.gnu.org/software/gsl/
[11]: #To-run-code-for-Calibration-or-Inference

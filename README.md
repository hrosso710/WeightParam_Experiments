# Reproducible Experiments for Polynomial Weight Parameterization in Meganet

This repository contains the code to reproduce the experiments based on polynomial weight parameterizations (monomial / Legendre) in neural ODE–style architectures implemented in **Meganet**.

All experiments are run from the main script:

- `runExperiment.m`

which uses

- `getPolynomialBasis.m`
- the **Meganet** library, which we renamed as the directory `Weight_Param_Meganet/` to include only parts of the library that are relevant to this manuscript.
- the dataset and setup script in `data/`

---

## 1. Repository Structure

At the top level, the repository has the following layout:

├── data
│   ├── NNERDS.mat
│   ├── param_range.txt
│   ├── setupNNERDS.m
│   ├── xall.txt
│   └── yall.txt
├── getPolynomialBasis.m
├── runExperiment.m
├── runExperiment.asv          # (MATLAB autosave, not needed to run)
└── Weight_Param_Meganet
    ├── activations/
    ├── augmentation/
    ├── data/
    │   ├── setupCDR.m
    │   └── setupDCR.m
    ├── examples/
    │   ├── GNvpro/
    │   └── slimTrain/
    ├── integrators/
    ├── KernelTypes/
    ├── layers/
    ├── loss/
    ├── matrixFree/
    ├── optimization/
    ├── regularization/
    ├── tests/
    ├── utils/
    ├── viewers/
    ├── Meganet.m
    └── README.md    # Original Meganet README

## 2. Requirements

- MATLAB (tested with 2024 and 2023 versions)
- Standard MATLAB toolboxes used by the Meganet library

No additional installation is needed beyond what is included in this repository.

Meganet is included as the Weight_Param_Meganet/ folder inside this repo; you do not need to install it separately.

## 3. Getting Started

1. Clone or download the repository 
2. Start MATLAB and set the current folder to the repository root where 'runExperiment.m' lives
3. Add required paths:
    % from the repo root
    addpath(genpath('Weight_Param_Meganet'));  % Meganet library and subfolders
    addpath('data');    % NNERDS dataset + setup script

## 4. Data Information

The default dataset used by runExperiment.m is data/NNERDS.mat and the corresponding setup script data/setupNNERDS.m.

The runExperiment.m script calls 
    [Yt, Ct, Yv, Cv, Ytest, Ctest] = setupNNERDS();

This function reads NNERDS.mat (and the additional helper files in data/) and returns:

    - Yt, Ct: training inputs / targets
    - Yv, Cv: validation inputs / targets
    - Ytest, Ctest: test inputs / targets

As long as you do not move the files in data/, no additional configuration is required.

For completeness: the original Meganet examples for the CDR and DCR datasets live in

Weight_Param_Meganet/data/
  ├── setupCDR.m
  └── setupDCR.m

These are not called directly by runExperiment.m, but are included for reference and for reproducing related Meganet examples.

## 5. Running the experiment

The main entry point is 
    runExperiment(dynamic, T, d, opti, basis)

with arguments:

    1. dynamic — choice of network dynamics: 'ResNN', 'antiSym-ResNN', 'leapfrog', 'hamiltonian'

    2. T — final time horizon for the ODE (e.g., T = 1.0).

    3. d — degree of the polynomial basis (integer, e.g., d = 3 or d = 4).

    4. opti — optimization method: 'sgd' — stochastic gradient descent / ADAM or 'GNvpro' — Gauss–Newton with variable projection

    5. basis — polynomial basis type: 'monomial' or 'Legendre' (if omitted or empty, defaults to 'monomial')
    
    
Internally, the script loads and splits the NNERDS data:

[Yt, Ct, Yv, Cv, Ytest, Ctest] = setupNNERDS();

Builds a two-block network:

    - Block 1: a dense layer mapping the input to nc channels
    - Block 2: a ResNN / Hamiltonian / Leapfrog block, depending on dynamic

Constructs the polynomial time-parameterization matrix A of size nt × (d+1) using monomials or Legendre polynomials (via getPolynomialBasis.m)

Sets up a regression loss and optimization problem (sgd or GNvpro).

Trains the network and prints mean/std/min/max relative error on train, validation, and test data

Produces plots of a 2×5 grid comparing model output vs. data for 10 components and asemilog plot showing optimality conditions, training loss, and validation loss across iterations.

From the MATLAB command window, with paths set as in Section 3:

## 6. Example Commands

% Example 1: ResNN with monomial basis, degree 3, GNvpro
runExperiment('ResNN', 1.0, 3, 'GNvpro', 'monomial');

% Example 2: ResNN with Legendre basis, degree 3, GNvpro
runExperiment('ResNN', 1.0, 3, 'GNvpro', 'Legendre');

% Example 3: Leapfrog dynamics with monomial basis, SGD optimization
runExperiment('leapfrog', 1.0, 3, 'sgd', 'monomial');

Feel free to adjust T (final time), d (polynomial degree), dynamic and opti to match the specific configurations reported in the manuscript.

At the top of runExperiment.m there is a flag:

doSave = 0;  % save file

## 7. Saving Results

At the top of runExperiment.m there is a flag:

    doSave = 0;  % save file

Set this to 1 if you want the script to copy the experiment script to a timestamped file, log output to a diary file, and save trained parameters (thOpt, WOpt) and training history (his) to a .mat file.

The results will be saved in the current directory using a timestamp-based filename.

## 8. Troubleshooting

“Undefined function or variable 'setupNNERDS'”
    - Make sure you have added the data folder to your path: addpath('data');

“Undefined function 'Meganet' / 'ResNN' / 'HamiltonianNN'”
    - Make sure you have added the Meganet folder: addpath(genpath('Weight_Param_Meganet'));

Different MATLAB version
    - The code relies on standard MATLAB features and the Meganet library. If you encounter version-specific issues, please mention your MATLAB version in any bug report.


## 9. Contact

If you have questions or problems reproducing the experiments, please contact:

Haley Rosso — <haley.rosso@emory.edu>

or open an issue on this repository.
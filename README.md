# SMILES-X
Autonomous molecular compounds characterization for small datasets without descriptors

On arXiv:

## What is it?
The **SMILES-X** is an autonomous pipeline which **find best neural architectures to predict a physicochemical property from molecular SMILES only** (see [OpenSMILES](http://opensmiles.org/opensmiles.html)). **No human-engineered descriptors are needed.**

## Which kind of dataset can I use with it?
The SMILES-X is dedicated to **small datasets (<5000 samples) of (SMILES, experimental property)**

## What can I do with it?
With the SMILES-X, you can:
* **Find best architectures** fitted to your small dataset via a Bayesian optimization.
* **Predict molecular properties** of a list of SMILES based on models ensembling.
* **Interpret** a prediction by visualizing the salient elements and/or substructures most related to an inferred property

## Usage

* Copy and paste **`SMILESX_utils.py`** to your working directory
* Use the following basic import to your jupyter notebook
```python
import pandas as pd

import SMILESX_utils
%matplotlib inline
```

### How to find the best architectures fitted to my dataset?
1. After basic libraries import, unfold your dataset
```python
validation_data_dir = "../validation_data/"
extension = '.csv'
data_name = 'FreeSolv' # FreeSolv, ESOL, Lipophilicity or your own dataset
prop_tag = 'expt' # which column corresponds to the property to infer in the *.csv file

sol_data = pd.read_csv(validation_data_dir+data_name+extension)
sol_data = sol_data[['smiles',prop_tag]] # reduce the data to (SMILES, property) sets
# If the column containing the SMILES has a different name, feel free to change it accordingly
```

2. Define architectural hyper-parameters bounds to be used for the neural architecture search
```python
dhyp_range = [int(2**itn) for itn in range(3,11)] # 
dalpha_range = [float(ialpha/10.) for ialpha in range(20,40,1)] # Adam's learning rate = 10^(-dalpha_range)

bounds = [
    {'name': 'lstmunits', 'type': 'discrete', 'domain': dhyp_range},  # number of LSTM units
    {'name': 'denseunits', 'type': 'discrete', 'domain': dhyp_range}, # number of Dense units
    {'name': 'embedding', 'type': 'discrete', 'domain': dhyp_range},  # number of Embedding dimensions
    {'name': 'batchsize', 'type': 'discrete', 'domain': dhyp_range},  # batch size per epoch during training
    {'name': 'lrate', 'type': 'discrete', 'domain': dalpha_range}     # Adam's learning rate 10^(-dalpharange) 
]
```
These bounds are used in the paper, but they can be tuned according to your dataset

3. Let the SMILES-X find the best architectures for the most accurate property predictions
```python
SMILESX_utils.Main(data=sol_data,        # provided data (SMILES, property)
                   data_name=data_name,  # dataset's name
                   data_units='',        # property's SI units
                   bayopt_bounds=bounds, # bounds contraining the Bayesian search of neural architectures
                   k_fold_number = 10,   # number of k-folds used for cross-validation
                   augmentation = True,  # SMILES augmentation
                   outdir = "../data/",  # directory for outputs (plots + .txt files)
                   bayopt_n_epochs = 10, # number of epochs for training during Bayesian search
                   bayopt_n_rounds = 25, # number of architectures to sample during Bayesian search 
                   bayopt_on = True,     # use Bayesian search
                   n_gpus = 1,           # number of GPUs to be used
                   patience = 25,        # number of epochs with no improvement after which training will be stopped
                   n_epochs = 100)       # maximum of epochs for training
```
Please refer to the **`SMILESX_utils.py`** for a detailed review of the options 

### How to infer a property on new data (SMILES)?
1. Just use
```python
pred_from_ens = SMILESX_utils.Inference(data_name=data_name, 
                                        smiles_list = ['CC','CCC','C=O'], # new list of SMILES to characterize
                                        data_units = '',
                                        k_fold_number = 3,                # number of k-folds used for inference
                                        augmentation = True,              # with SMILES augmentation
                                        outdir = "../data/")
```

It returns a table of SMILES with their inferred property (mean, standard deviation) determined by models ensembling

### How to interpret a prediction?
1. Just use
```python
SMILESX_utils.Interpretation(data=sol_data, 
                             data_name=data_name, 
                             data_units='', 
                             k_fold_number = 3,
                             k_fold_index = 0,           # model id to use for interpretation
                             augmentation = True, 
                             outdir = "../data/", 
                             smiles_toviz = 'ClC1=C(Cl)[C@]2(Cl)[C@H]3[C@H]4C[C@H]([C@@H]5O[C@@H]54)[C@H]3[C@@]1(Cl)C2(Cl)Cl', 
                             font_size = 15,             # plots font parameter
                             font_rotation = 'vertical') # plots font parameter
```

Returns:
1. 1D attention map on individual tokens
![1d_attention_map](/images/Interpretation_1D_FreeSolv_SAMPL_seed_17730.png)

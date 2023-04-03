# SeqPredNN

Deep feed-forward neural network for predicting amino acid sequences from protein conformations.

## Table of Contents

1. ![Requirements](##Requirements)
2. ![Usage](##Usage)
      1. ![Installing dependencies](###Installing-dependencies)
      2. ![Predicting protein sequences](###Predicting-protein-sequences)


## Requirements

* Python 3.9
* Pytorch
* Numpy
* SciKit-Learn
* Matplotlib
* Scipy
* Biopython

## Usage

### Installing dependencies

We recommend using ![conda](https://docs.conda.io/projects/conda/en/stable/user-guide/install/index.html) to install the required python packages in a contained environment:

1. Import the SeqPredNN environment using the SeqPredNN_environment.yml file
        conda env create -n SeqPredNN -f SeqPRedNN_environment.yml
2. Activate the conda environment before using SeqPredNN
        conda activate SeqPredNN

### Predicting protein sequences

![Prediction process flowchart](/prediction_diagram.png)

1.  Prepare input files

      - Predicting an amino acid sequence for a set of protein structures requires:
        
           1. a directory containing the .pdb format files of your protein structures
        
           2. a comma-separated list of protein names, pdb filepaths in the abovementioned directory, and protein chain IDs for each protein chain e.g. the row for chain B of protein 1HST in the file /examples/example_pdb_directory/1hst.pdb.gz would read "1HST, 1hst.pdb.gz, B"
      
           3. The neural network parameters of the trained sequence prediction model
      
      - Examples of a ![chain list](/examples/chain_list.csv) and ![PDB directory](/examples/example_pdb_directory) are given in ![/examples/](/example)
      
      - We vaildated SeqPredNN using the ![pretrained SeqPredNN model parameters] and recommend you use these parameters to generate protein sequences

2.  Generate structural features for your protein structures using ![featurise.py](/SeqPredNN)

        python SeqPredNN/featurise.py -gm -o example_features examples/chain_list.csv examples/example_pdb_directory
 
    - The `-gm` argument indicates that the structure files are gzipped and should be uncompressed before they are parsed (`-g`), and that modified amino acids should be converted to the appropriate unmodified standard amino acid (`-m`)
    - The `-o` argument indicates the directory where the structural features will be saved (in this case the features will be saved in `example_features/`)
    - There are two positional arguments:
      1. the chain list
      2. the PDB directory
    - For additional command line arguments run 
    
            python SeqPredNN/featurise.py --help

2. Predict amino acid sequences using ![predict.py](/SeqPredNN)

       python SeqPredNN/predict.py -p example_features example_features/chain_list.txt pretrained_model/pretrained_parameters.pth
 
    - prediction-only mode `-p` only predicts sequences and does not evaluate the model by comparing predicted sequences with the original sequence
    - There are three positional arguments:
      1. the directory where the features are saved (here example_features)
      2. a newline-seperated text file listing all the protein chains to be predicted (chain_list.txt lists all the featurised chains. It is automatically generated in the feature directory)
      3. the neural network parameters (here ![pretrained_model/pretrained_parameters.pth](/pretrained_model))
 
### Training your own model:

![Train process flowchart](/train_diagram.png)

1. Download the PDB files of the structures in your training dataset - https://www.wwpdb.org/ftp/pdb-ftp-sites
2. Generate structural features for the proteins using ![featurise.py](/SeqPredNN)
    - see ![**Predicting protein sequences**](###Predicting-protein-sequences) for more details
3. Train the model using `train_model.py`

       train_model.py -r 0.9 -t test_chains.txt -o my_model -e 200 feature_directory balanced

    - The train ratio (-r) is the fraction of residues assigned to the training dataset. The remaining residues are assigned to a validation set used to evaluate the model during training
    - The test chain file (-t) specifies which chains should be excluded from the training and validation datasets so that they can be used for independent evaluation of the model. The test chain file must be in the same format as examples/chain_list.txt.
    - (-e) is used to specify the number of epochs for training
    - The balanced/unbalanced keyword specifies the sampling mode. "unbalanced" sampling partitions all the residues in the features into the training and validation datasets. "balanced" sampling undersamples the residues so that each of the 20 amino acid classes occur the same number of times in the dataset.
4. Test your model using `prediction.py`
                
       prediction.py - o predictions feature_directory test_chains.txt my_model/model_parameters.pth
          
   * Evaluation output:
     * A classification report with precision, recall and f1-score for each amino acid class
     * The top K accuracy of the predictions for each amino acid class
     * 3 confusion matrices (unnormalised, normalised by prediciton and normalised by true residue)
     * For each chain in the test set:
       * The predicted sequence
       * The probabilities for each amino acid class produced by the model for each preducted residue
       * A classification report
       * Cross-entropy loss for each predicted residue

## Pretrained model 

The pretrained model was trained using the chains in pretrained_model/40_res3_nobrks.pisces. The dataset consists of 18914 chains with less than 40% sequence similarity, resolution < 3 angstrom, no chain breaks, length of 40-10000 residues, R-factor < 0.3 and only X-ray crystallography structures. It was generated by the [pisces server](https://dunbrack.fccc.edu/pisces/) on 2021-11-19. Some proteins in 40_res3_nobrks.pisces were missing from the PDB snapshot used to train the model - these chains were excluded from the training data and are listed in pretrained_model/excluded.txt. We excluded a random test set (pretrained_model/random_test_set.txt) of 10% of the chains from training. 

## Licence
This software and code is distributed under a GNU General Public License V3

## Poster

![SeqPredNN Conference Poster](/SeqPredNN_Poster.png)

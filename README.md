# AiZynthFinder Documentation

## General Description
- AizynthFinder is a cutting-edge tool for retrosynthetic planning in computational chemistry and medicinal chemistry.
- The tool employs a Monte Carlo tree search algorithm to recursively decompose a target molecule into purchasable precursors. This tree search process is guided by a policy that recommends potential precursors using a neural network trained on a comprehensive library of known reaction templates.
- AizynthFinder facilitates the discovery of synthetic routes, optimizing the process of chemical synthesis
- An introduction video can be found: https://youtu.be/r9Dsxm-mcgA
## Installation
### System Requirements
Before you begin, ensure you have met the following prerequisites:
  - __Supported Platforms__ : Linux, Windows, or macOS. Ensure dependencies are supported on these platforms.
  - __Python Version__: Anaconda or Miniconda with Python version 3.9 to 3.11 installed.
    - You can download Anaconda from [Anaconda Distribution](https://www.anaconda.com/products/distribution).
    - Alternatively, download Miniconda from [Miniconda Documentation](https://docs.conda.io/en/latest/miniconda.html).
  - __Platform Specifics__:
     - The tool has been primarily developed and tested on a Linux platform.
     - It has also been tested on Windows 10 and macOS Catalina. 
### Installation Steps for End-Users
1. **Create Conda Environment**:
   - Open a console or an Anaconda prompt and execute the following command to create a new Conda environment named `aizynth-env` with Python version 3.9 to 3.10:
   ```bash
   conda create -n aizynth-env "python>=3.9,<3.11"
2. **Activate the Environment**
   - Activate the newly created Conda environment `aizynth-env`:
   ```bash
   conda activate aizynth-env
3. **Install AiZynthFinder**
   - Install the AiZynthFinder package from PyPI.
   - For the full package including all functionalities, run:
    ```bash
    python -m pip install aizynthfinder[all]
    ```
    - Alternatively, for a smaller package without all functionalities, run:
    ```bash
    python -m pip install aizynthfinder
## Usage
The tool installs `aizynthcli` and `aizynthapp` as interfaces to the algorithm.
To use the tool, the following files are required:
  - A stock file
  - A trained expansion policy network
  - A trained filter policy network (optional)
       
Such files can be downloaded using:
```bash
download_public_data my_folder
```
```bash
# This will download the data to your current directory
download_public_data .
```
It will download a public available model based on USPTO and a stock collection from ZINC database. The `config.yml` file can be directly used with both interfaces provided by the package. 
### Command-line interface
It offers capabilities for conducting tree searches on batches of molecules.

It can be executed like:
```bash
aizynthcli --config config.yml --smiles smiles.txt
```
Here, `config.yml` includes configurations like paths to policy models and stocks, while `smiles.txt` is a text file containing SMILES, each on a separate line.

To view all available arguments, use the `-h` flag with `aizynthcli`, as shown below:
```bash
aizynthcli -h
```
This command provides the following options:
```bash
usage: aizynthcli [-h] --smiles SMILES --config CONFIG
                  [--policy POLICY [POLICY ...]]
                  [--filter FILTER [FILTER ...]]
                  [--stocks STOCKS [STOCKS ...]]
                  [--output OUTPUT]
                  [--log_to_file]
                  [--nproc NPROC]
                  [--cluster]
                  [--route_distance_model ROUTE_DISTANCE_MODEL]
                  [--post_processing POST_PROCESSING [POST_PROCESSING ...]]
                  [--pre_processing PRE_PROCESSING]
                  [--checkpoint CHECKPOINT]

options:
  -h, --help            show this help message and exit
  --smiles SMILES       specify the target molecule SMILES or the path to a file containing SMILES
  --config CONFIG       specify the configuration file name
  --policy POLICY [POLICY ...]
                        specify the expansion policy or policies to use
  --filter FILTER [FILTER ...]
                        specify the filter or filters to use
  --stocks STOCKS [STOCKS ...]
                        specify the stocks to use
  --output OUTPUT       specify the output file name (JSON or HDF5)
  --log_to_file         enable detailed logging to file
  --nproc NPROC         specify the number of processes for parallel execution
  --cluster             enable automatic clustering
  --route_distance_model ROUTE_DISTANCE_MODEL
                        specify the ML model for calculating route distances for clustering
  --post_processing POST_PROCESSING [POST_PROCESSING ...]
                        specify modules for post-processing tasks
  --pre_processing PRE_PROCESSING
                        specify a module for pre-processing tasks
  --checkpoint CHECKPOINT
                        specify the path to a checkpoint file
```
By default:
- All stocks are selected if none are specified.
- The first expansion policy is selected if none are specified.
- All filter policies are selected if none are specified on the command-line.

To perform a tree search for a single molecule, you can specify it directly on the command line within quotes
```bash
aizynthcli --config config.yml --smiles "COc1cccc(OC(=O)/C=C/c2cc(OC)c(OC)c(OC)c2)c1"
```
Ensure to replace config.yml with the path to your configuration file and adjust the SMILES string ("COc1cccc(OC(=O)/C=C/c2cc(OC)c(OC)c(OC)c2)c1") to match the SMILES of your target molecule."

__Specification of Output__

When using `aizynthcli` with multiple SMILES, the tool generates either a JSON or HDF5 file, typically named `output.json.gz`. This file is structured to be easily read into a pandas DataFrame for convenient data manipulation, as demonstrated below:
```python
import pandas as pd

# Read the output JSON or HDF5 file into a pandas DataFrame
data = pd.read_json("output.json.gz", orient="table")
```
The `output.json.gz` file includes statistical information about the tree search and the top-ranked routes, each represented as JSON objects corresponding to each target compound processed. It will contain the following columns:

  - **target**: The target SMILES.
  - **search_time**: Total search time in seconds.
  - **first_solution_time**: Time elapsed until the first solution was found.
  - **first_solution_iteration**: Number of iterations completed until the first solution was found.
  - **number_of_nodes**: Number of nodes in the search tree.
  - **max_transforms**: Maximum number of transformations for all routes in the search tree.
  - **max_children**: Maximum number of children for a search node.
  - **number_of_routes**: Number of routes in the search tree.
  - **number_of_solved_routes**: Number of solved routes in the search tree.
  - **top_score**: Score of the top-ranked route (default to MCTS reward).
  - **is_solved**: Indicates if the top-ranked route is solved.
  - **number_of_steps**: Number of reactions in the top-ranked route.
  - **number_of_precursors**: Number of starting materials.
  - **number_of_precursors_in_stock**: Number of starting materials available in stock.
  - **precursors_in_stock**: Comma-separated list of SMILES of starting materials in stock.
  - **precursors_not_in_stock**: Comma-separated list of SMILES of starting materials not in stock.
  - **precursors_availability**: Semi-colon-separated list indicating the availability of starting materials.
  - **policy_used_counts**: Dictionary showing the total number of times each expansion policy has been used.
  - **profiling**: Profiling information from the search tree, including expansion model calls and reactant generation.
  - **stock_info**: Dictionary detailing stock availability for each starting material in all extracted routes.
  - **top_scores**: Comma-separated list of scores of the extracted routes (default to MCTS reward).
  - **trees**: A list of the extracted routes represented as dictionaries.

Additionally, if a checkpoint file path is specified during the execution of `aizynthcli`, a file named `checkpoint.json.gz` is created. This file logs processed SMILES along with their respective results, each listed on a separate line.

However, when providing a single SMILES string to `aizynthcli`, the tool displays statistical data directly in the terminal. The top-ranked routes are saved into a default JSON file named `trees.json`.

To generate images of the top-ranked routes for the first target compound, you can use the following example code snippet:
```python
import pandas as pd
from aizynthfinder.reactiontree import ReactionTree

# Read the output JSON or HDF5 file into a pandas DataFrame
data = pd.read_json("output.json.gz", orient="table")

# Access all trees for all compounds
all_trees = data.trees.values

# Select trees for the first target compound
trees_for_first_target = all_trees[0]

# Create images for each route of the first target compound
for itree, tree in enumerate(trees_for_first_target):
    imagefile = f"route{itree:03d}.png"
    ReactionTree.from_dict(tree).to_image().save(imagefile)
```
### Graphical User Interface

To perform a tree search on a single compound using a GUI in a Jupyter notebook with AiZynthFinder, you can follow these steps:

1. Make sure Jupyter Notebook is installed. If not, install it using:
   ```bash
   pip install notebook
  ```
2. Launch Jupyter Notebook by running the following command in your terminal:
  ```bash
   jupyter notebook
  ```
3. Browse to an existing notebook or create a new one.
4. Add these lines to the first cell in the notebook.
   ```python
   from aizynthfinder.interfaces import AiZynthApp
   app = AiZynthApp("/path/to/configfile.yaml")
   ```
5. Executed the code in the cell (press Ctrl+Enter) and a simple GUI will appear.
   
! /Users/Usuario/Pictures/Screenshots/Captura de pantalla 2024-06-28 123109.png

<h1 align="center">Privacy Enhancement for Text via Risk-oriented Explainability (PETRE)</h1>

<p align="center">
  <img src="https://img.shields.io/badge/Windows%2011-Working-ok" alt="Working on Windows 11"/>
  <img src="https://img.shields.io/badge/Linux_based-Compatible_but_not_tested-lightgrey" alt="Compatible but not tested on Linux-based systems"/>
  <img src="https://img.shields.io/badge/Windows%2010-Compatible_but_not_tested-lightgrey" alt="Compatible but not tested on Windows 10"/>
</p>

This repository contains the code and data for the **privacy enhancement for text via risk-oriented explainability** (PETRE) method presented in **B. Manzanares-Salor, D. Sánchez, Enhancing text anonymization via re-identification risk-based explainability, Submitted, 2024**.

Experimental data was extracted from the [bootstrapping-anonymization](https://github.com/anthipapa/bootstrapping-anonymization) repository, corresponding to the publication [A. Papadopoulou, P. Lison, L. Øvrelid, I. Pilán, Bootstrapping Text Anonymization Models with Distant Supervision, Proceedings of the Thirteenth Language Resources and Evaluation Conference, pages 4477–4487, Marseille, France, 2022](https://aclanthology.org/2022.lrec-1.476/). The exact data files utilized in the experiments are located in the [examples](examples) folder.

## Table of Contents
* [Project structure](#project-structure)
* [Install](#install)
* [Usage](#usage)
  * [CLI](#cli)
  * [PETRE class](#petre-class)
* [Configuration](#configuration)
* [Results](#results)
* [Examples](#examples)


# Project structure
```
Privacy Enhancement for Text via Risk-oriented Explainability (PETRE)
│   README.md                               # This README
│   petre.py                                # Python program including the PETRE class, which can be used from Python code or CLI
│   requirements.txt                        # File generated with Conda containing all the required Python packages with specific versions
└───examples                                # Folder with examples
    │   config.json                         # Example configuration
    │   Wiki553.json                        # Panda's dataframe with 553 individuals, documents to protect and background knowledge
    │   Annotations_St.NER3.json            # Anonymization annotations from Standford NER3
    │   Annotations_St.NER4.json            # Anonymization annotations from Standford NER4
    │   Annotations_St.NER7.json            # Anonymization annotations from Standford NER7
    │   Annotations_Presidio.json           # Anonymization annotations from Microsoft Presidio
    │   Annotations_Word2Vec_t=0.25.json    # Anonymization annotations from the Word2Vec-based method
    │   Annotations_k_anonymity_Greedy.json # Anonymization annotations from the k-anonymity greedy method
    │   Annotations_k_anonymity_Random.json # Anonymization annotations from the k-anonymity random method
    │   Annotations_Manual.json             # Anonymization annotations from human annotators
```

# Install
Our implementation uses [Python 3.9.19](https://www.python.org/downloads/release/python-3919/) as programming language and [Conda](https://docs.conda.io/en/latest/) 24.1.2 for package management. All used packages and their respective versions are listed in the [requirements.txt](requirements.txt) file. 

To be able to run the code, follow these steps:
1. Install Conda if you haven't already.
2. Download this repository.
3. Open a terminal in the repository path.
4. Create a new Conda environment using the following command (channels included for ensuring that specific versions can be installed):
```console
conda create --name ENVIRONMENT_NAME --file requirements.txt -c conda-forge -c spacy -c pytorch -c nvidia -c huggingface -c numpy -c pandas
```
5. Activate the just created Conda environment using the following command:
```console
conda activate ENVIRONMENT_NAME
```
Continue with the steps of the [Usage section](#usage).

This has been tested in Windows 11 operating system, but should be compatible with Linux-based and Windows 10 systems.

# Usage
The PETRE method is implemented in the [petre.py](petre.py) script. They can be executed via [CLI](#cli) (Command Line Interface) or by importing the [PETRE class](#petre-class) directly into your Python code. The following sections provide detailed instructions for both approaches. Additionally, both methods offer configuration options (details on how in each subsection), which are described in the [Configuration section](#configuration).

## CLI
The CLI implementation only requires to pass as argument the path to a JSON configuration file. This file must contain a dictionary with the mandatory configurations, and can also contain optional configurations (see [Configuration section](#configuration)).

For example, for using the configuration file [examples/config.json](examples/config.json), run the following command:
```console
python petre.py examples/config.json
```

## PETRE class
You can replicate the CLI behavior in a Python file by importing the `PETRE` class from the [petre.py](petre.py) script, instanciating the class and calling to its `run` method. The constructor requires the mandatory configurations as arguments, and also accepts optional configurations (see [Configuration section](#configuration)). Moreover, any of these configurations can be later modified by calling the `set_configs` method. During the execution, multiple loggings indicate the current block within the PETRE process. These loggings can be disabled by passing `verbose=False` as argument to the `run` method.

Here is a Python snippet that demonstrates how to run PETRE for all the starting anonymizations in the [examples](examples) folder:
```python
import os
from petre import PETRE

# Declare PETRE mandatory settings (placeholders for output and data paths) and some optional settings
petre = PETRE(output_base_folder_path="outputs/Wiki553",
              data_file_path="examples/Wiki553.json",
              individual_name_column="name",
              original_text_column="original",
              starting_anonymization_path="To Be Defined",
              tri_pipeline_path="./TRI_Pipeline",
              ks=[2,3,5,7,10])

# Evaluate all the starting anonymizations in the "examples" folder
starting_anonymizations_folder = "examples"
for annotations_file_name in os.listdir(starting_anonymizations_folder):
    # Only consider annotations files
    if annotations_file_name.startswith("Annotations"):
        # Set new starting_anonymization_path configuration
        starting_anonymization_path = os.path.join(starting_anonymizations_folder, annotations_file_name)
        petre.set_configs(starting_anonymization_path=starting_anonymization_path)
        
        # Run PETRE for this starting anonymization
        petre.run(verbose=True)
```

## Configuration
In the following, we specify the configurations available for our implementation, as [CLI](#cli) (with the JSON file) or as [PETRE class](#petre-class) (with the constructor or `set_configs` method). For each configuration, we specify the name, type, if it is mandatory or optional (i.e., has a default value) and description.

* **Mandatory configurations**:
  * **output_base_folder_path | String | MANDATORY**: Determines the base folder were results will be stored (see [Results section](#results)). The folder will be created if it does not exist.
  * **data_file_path | String | MANDATORY**: Path to the data file containing the original documents to protect. The file is expected to define a Pandas dataframe stored in JSON or CSV format containing at least two columns:
    * *Individual name*: Column with the names of all the individuals, titled as defined in the `individual_name_column` setting.
    * *Original text*: Column with the original document to protect. It is expected that all individuals have a document to protect.
    Additional columns will have no effect on the behaviour of the code.
    Example of a dataframe with **three** individuals, each of them with an original document to protect:

        | name            | original                                                        |
        |-----------------|-----------------------------------------------------------------|
        | Joe Bean        | Bean received funding from his family to found UnderPants.      |
        | Ebenezer Lawson | Lawson, born in Kansas, has written multiple best-sellers.      |
        | Ferris Knight   | After a long race, Knight managed to obtain the first position. |  

  * **individual_name_column | String | MANDATORY**: Name of the dataframe column corresponding to the individual name. In the previous example, it will be `name`.
  * **original_text_column | String | MANDATORY**: Name of the column corresponding to the original document to protect for each individual. In the previous example, it will be `original`.
  * **starting_anonymization_path | String | MANDATORY**: Path to the annotations to be used as starting anonymization. Annotations must be stored in JSON formatted dictionary, with individual's `name` (defined in the `data_file_path`) as key and a list of the masked spans as value. It is expected that all individuals have annotations.
  * **tri_pipeline_path | String | MANDATORY**: Path to the [Transformers' pipeline](https://huggingface.co/docs/transformers/main_classes/pipelines) containing the text re-identification (TRI) model and tokenizer. Refer to the [Text Re-Identification repository](https://github.com/BenetManzanaresSalor/TextRe-Identification) for the creation of this pipeline. If the dataframe at `data_file_path` contains, apart from the original documents and individuals' names, a column with the background knowledge for each individual, it can be leveraged for the creation of this pipeline (more details in the [Examples](#examples)). *NOTE: You can also use a path to a pipeline from the [HuggingFace models' repository](https://huggingface.co/models).*
  * **ks | List of integers | MANDATORY**: Sorted list of the $k$ values to be used by PETRE for incremental execution. For instance, with `ks=[2,3]`, annotations generated for $k=2$ will serve as a starting point for ensuring $k=3$. This approach helps reduce execution time, as annotations for $k=X$ are assumed to be a superset of those for $k<X$. Since this $k$ values refer to probabilistic $k$-anonymity, $k$ must be greater than 1 in any case, with $k=1$ providing no protection and $k=2$ offering the mininum protection.

* **Optional configurations**:
  * **mask_text | String | Default=""**: Text by which each annotated term will be replaced prior to the re-identification attack. It might influence the accuracy of the `TRI_Pipeline`.
  * **use_mask_all_instances | Boolean | Default=true**: Once PETRE finds a term as the most disclosive, whether to mask all instances of that term in the whole document. If it is `false`, only the instance found to be most revealing will be masked. Although this may lead to unnecessary masking, it can significantly reduce PETRE's runtime.

## Results
After execution of PETRE (both from CLI or Python code), a folder with same name as the starting anonymization file (defined in `starting_anonymization_path`) will be created in the `output_base_folder_path`. This folder will contain:
* **Annotations_PETRE_k=X.json**: File containing the annotations created by PETRE for $k=X$. A separate file will be created for each $k$ specified in the `ks` setting.
* **Ranks_k=X.csv**: Considering the annotations created by PETRE for $k=X$, this file lists the re-identification ranks for each individual's document, organized alphabetically by individual name. The rank indicates the position within the re-identification probability distribution, where a rank of 1 represents the highest predicted probability for re-identification, signifying the maximum risk. A separate file will be created for each $k$ specified in the `ks` setting.


## Examples
In the [examples](examples) folder we provide: a basic configuration file [config.json](examples/config.json), the [Wiki553.json](examples/Wiki553.json) Pandas dataframe with data from the [bootstrapping-anonymization](https://github.com/anthipapa/bootstrapping-anonymization) repository, and multiple starting anonymizations' annotations. The [config.json](examples/config.json) is set up to use the [Wiki553.json](examples/Wiki553.json) for original documents and individuals' names, a re-identification pipeline located at `./TRI_Pipeline` (not included here due to the 100MB storage limit), multiple $k$ values ranging from 2 to 10, and [Annotations_k_anonymity_Greedy.json](examples/Annotations_k_anonymity_Greedy.json) as starting anonymization. This is the most strict starting anonymization, requiring less enhancement from PETRE and thus minimizing this example's runtime (see **our paper** for more details).

The [Wiki553.json](examples/Wiki553.json) contains, apart from the "name" and "original" columns, a column named "background_knowledge" with the background knowledge for each individual and a column named "dev" with a 10% development subset of the original documents anonymized with [spaCy NER](https://spacy.io/api/entityrecognizer). In this way, this dataframe can be directly used for the creation of the `TRI_Pipeline` using the [Text Re-Identification repository](https://github.com/BenetManzanaresSalor/TextRe-Identification). Use the filepath for the `data_file_path` setting, "name" for the `individual_name_column` setting, "background_knowledge" for the `background_knowledge_column` setting, "dev" for the `dev_set_column_name` setting and "No" for the `finetuning_sliding_window` setting (so it is trained to re-identify text at sentence level instead of with sliding window).

Feel free to modify [config.json](examples/config.json) to use other dataframes, re-identification pipelines, $k$ values or starting anonymizations. If adjusting settings other than `starting_anonymization_path`, we recommend also updating the `output_base_folder_path` setting to prevent overwriting results.
# BioDEX

## Installation
Create the conda environment:

    conda create -n biodex python=3.9
    conda activate biodex
    pip install -r requirements.txt
    pip install .

## The BioDEX resource

The report fields are described in detail [here](https://open.fda.gov/apis/drug/event/searchable-fields/).

## Dataset Creation
All our dataset can be downloaded from HuggingFace.

The code to produce the ICSR-Extraction dataset is located in `data_creation/icsr_extraction`.
The code to produce the raw resource will be released soon.

## Tasks

Code to train and evaluate on the ICSR-Extraction task is located in `tasks/icsr_extraction`. Further directions can be found in the relevant folder.
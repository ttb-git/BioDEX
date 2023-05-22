# BioDEX: Large-Scale Biomedical Adverse Drug Event Extraction for Real-World Pharmacovigilance.

This is the official repository for the [BioDEX paper](todo).

BioDEX is a raw resource for drug safety monitoring that bundles full-text and abstract-only PubMed papers with drug safety reports. These reports contain structured information about an Adverse Drug Events (ADEs) described in the papers, and are produced by medical experts in real-world settings.

BioDEX contains 19k full-text papers, 65k abstracts, and over 256k associated drug-safety reports.

<!-- We hope that our resource paves the way for (semi-)automated ICSR reporting systems, which one day could aid humans to perform drug safety monitoring. Additionally, we believe our task is a good resource to train and evaluate the biomedical capabilities of Large Language Models. -->

<!-- 
BioDEX is created by combining the following resources:
- [PubMed Medline](https://www.nlm.nih.gov/bsd/difference.html): Distribution of PubMed article metadata and abstracts.
- [PubMed Central](https://www.ncbi.nlm.nih.gov/pmc/tools/openftlist/): Distribution of full-text PubMed articles.
- [FAERS](https://www.fda.gov/drugs/surveillance/questions-and-answers-fdas-adverse-event-reporting-system-faers): Distribution of Individual Case Sefety Reports. -->

<!-- A preliminary write-up including initial results is included in `supplementary_material/`. A description of the dataset fields is given in `supplementary_material/BioDEX_Dataset_Card.pdf`. -->



## Installation
Create the conda environment and install the code: 

    conda create -n biodex python=3.9
    conda activate biodex
    pip install -r requirements.txt
    pip install .

## Load the raw resource
```python
import datasets

# load the raw dataset
dataset = datasets.load_dataset("FAERS-PubMed/raw_dataset")['train']

print(len(dataset)) # 65,648

# investigate an example
article = dataset[1]['article']
report = dataset[1]['reports'][0]

print(article['title'])    # Case Report: Perioperative Kounis Syndrome in an Adolescent With Congenital Glaucoma.
print(article['abstract']) # A 12-year-old male patient suffering from congenital glaucoma developed bradycardia, ...
print(article['fulltext']) # ...
print(article['fulltext_license']) # CC BY

print(report['patient']['patientsex']) # 1
print(report['patient']['drug'][0]['activesubstance']['activesubstancename']) # ATROPINE SULFATE
print(report['patient']['drug'][0]['drugadministrationroute']) # 040
print(report['patient']['drug'][1]['activesubstance']['activesubstancename']) # MIDAZOLAM
print(report['patient']['drug'][1]['drugindication']) # Anaesthesia
print(report['patient']['reaction'][0]['reactionmeddrapt'])  # Kounis syndrome
print(report['patient']['reaction'][1]['reactionmeddrapt'])  # Hypersensitivity
```

Optional, use our code to parse the raw resource into Python objects for easy manipulation
```python
import datasets
from src.utils import get_matches

# load the raw dataset
dataset = datasets.load_dataset("FAERS-PubMed/raw_dataset")['train']
dataset = get_matches(dataset)

print(len(dataset)) # 65,648

# investigate an example
article = dataset[1].article
report = dataset[1].reports[0]

print(article.title)    # Case Report: Perioperative Kounis Syndrome in an Adolescent With Congenital Glaucoma.
print(article.abstract) # A 12-year-old male patient suffering from congenital glaucoma developed bradycardia, ...
print(article.fulltext) # ...
print(article.fulltext_license) # CC BY

print(report.patient.patientsex) # 1
print(report.patient.drug[0].activesubstance.activesubstancename) # ATROPINE SULFATE
print(report.patient.drug[0].drugadministrationroute) # 040
print(report.patient.drug[1].activesubstance.activesubstancename) # MIDAZOLAM
print(report.patient.drug[1].drugindication) # Anaesthesia
print(report.patient.reaction[0].reactionmeddrapt)  # Kounis syndrome
print(report.patient.reaction[1].reactionmeddrapt)  # Hypersensitivity
```

## Load the Report-Extraction dataset
```python
import datasets

# load the report-extraction dataset
dataset = datasets.load_dataset("FAERS-PubMed/BioDEX-ICSR")

print(len(dataset['train']))        # 9,624
print(len(dataset['validation']))   # 2,407
print(len(dataset['test']))         # 3,628

example = dataset['train'][0]

print(example['fulltext_processed'][:1000], '...') # TITLE: # SARS-CoV-2-related ARDS in a maintenance hemodialysis patient ...
print(example['target']) # serious: 1 patientsex: 1 drugs: ACETAMINOPHEN, ASPIRIN ...
```

<!-- ## TODO Use our Report-Extraction model
```python
from transformers import AutoModelForSeq2SeqLM, AutoTokenizer

model = AutoModelForSeq2SeqLM.from_pretrained("FAERS-PubMed/flan-t5-large-report-extraction")
tokenizer = AutoTokenizer.from_pretrained("FAERS-PubMed/flan-t5-large-report-extraction")

# TODO
input = """TODO"""
input = tokenizer.encoder(input)

output = model.predict(**input)

``` -->

## Overview of this repository
This repository is structured as follows:
- `analysis/` contains the data and notebooks to reproduce all plots in the paper.
- `src/` contains all code to represent the data objects and calculate the metrics.
- `data_creation/` contains the code to create the Report-Extraction dataset starting from the raw resource. Code to create the raw resource from scratch from will be released soon.
- `task/icsr_extraction/` contains the code to train and evaluate models for the Report-Extraction task.

### The Report-Extraction Task
All code for this task is located in `task/icsr_extraction/`.

Train an encoder-decoder model:
```
# train encoder-decoder models (e.g. T5)
$ cd tasks/icsr_extraction
$ python run_encdec_for_icsr_extraction.py `python ../config_to_cmd.py config_t5.yaml`
```

Train a decoder-only model (still a work in progress):
```
# train decoder-only models (e.g. GPT2)
$ cd tasks/icsr_extraction
$ python run_decoder_for_icsr_extraction.py `python ../config_to_cmd.py config_gpt2.yaml`
```

You can either pass command line arguments directly or save them in a config and parse the config to command line by running `` `python ../config_to_cmd.py config_t5.yaml` ``. Don't forget the backticks to first execute this command. Set the `text_column` field to either `abstract` or `fulltext_processed` to train a model on only the abstract or the entire paper respectively.



<!-- More about our evaluation procedure in the paper.

Example input:
```
TITLE: A Case of Pancytopenia with Many Possible Causes: How Do You Tell Which is the Right One? 

ABSTRACT: Systemic lupus erythematosus (SLE) often presents with cytopenia(s); however, pancytopenia is found less commonly, requiring the consideration of possible aetiologies other than the primary disease. The authors describe the case of a female patient with a recent diagnosis of SLE admitted through the Emergency Department with fever of unknown origin and severe pancytopenia. She was medicated with prednisolone, hydroxychloroquine, azathioprine, amlodipine and sildenafil. Extensive investigation suggested azathioprine-induced myelotoxicity. However, the patient was found to have a concomitant cytomegalovirus (CMV) infection, with oral lesions, positive CMV viral load as well as the previously described haematological findings. Pancytopenia is always a diagnostic challenge, with drug-induced myelotoxicity, especially secondary to azathioprine, being a rare aetiology. This report reiterates the importance of the differential diagnosis of pancytopenia, especially in immunosuppressed patients with increased risk for opportunistic infections. The possibility of multiple aetiologies for pancytopenia in the same patient should be considered.Azathioprine-induced myelotoxicity is dose-dependent and pancytopenia is a rare form of presentation.Opportunistic infections should always be considered as a cause of cytopenias when immunosuppression is present. 

TEXT: INTRODUCTION Systemic lupus erythematosus (SLE) is a chronic inflammatory … [truncated]
```

Example output:
```
serious: yes 
patientsex: female 
drugs: AMLODIPINE BESYLATE, AZATHIOPRINE, HYDROXYCHLOROQUINE, PREDNISOLONE, SILDENAFIL 
reactions: Bone marrow toxicity, Cytomegalovirus infection, Cytomegalovirus mucocutaneous ulcer, Febrile neutropenia, Leukoplakia, Odynophagia, Oropharyngeal candidiasis, Pancytopenia, Product use issue, Red blood cell poikilocytes present, Vitamin D deficiency
``` -->

<!-- ## Dataset Creation
All our datasets are available on the HuggingFace hub. We are in the process of releasing all code to reproduce these datasets. 

| Task            | HuggingFace dataset                                                                                                         | Leaderboard   | Code to reproduce dataset         |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------|---------------|-----------------------------------|
| raw resource    | [FAERS-PubMed/raw_dataset](https://huggingface.co/datasets/FAERS-PubMed/raw_dataset)                                        | /             | `data_creation/raw` (coming soon) |
| ICSR-Extraction | [FAERS-PubMed/BioDEX-ICSR](https://huggingface.co/datasets/FAERS-PubMed/BioDEX-ICSR/viewer/FAERS-PubMed--BioDEX-ICSR/train) | (coming soon) | `data_creation/icsr_extraction`   |
| ICSR-QA         | (coming soon)                                                                                                               | (coming soon) | (coming soon)                     | -->

## Limitations
See section 9 of the [BioDEX paper](todo) for limitations and ethical considerations.

## Contact
Open an issue on this GitHub page or email `karel[dot]doosterlinck[at]ugent[dot].be` and preferrably include "[BioDEX]" in the subject.

## License
Coming soon.

## Citation
Coming soon.

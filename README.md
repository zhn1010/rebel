[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/rebel-relation-extraction-by-end-to-end/relation-extraction-on-nyt)](https://paperswithcode.com/sota/relation-extraction-on-nyt?p=rebel-relation-extraction-by-end-to-end)
[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/rebel-relation-extraction-by-end-to-end/relation-extraction-on-conll04)](https://paperswithcode.com/sota/relation-extraction-on-conll04?p=rebel-relation-extraction-by-end-to-end)
[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/rebel-relation-extraction-by-end-to-end/joint-entity-and-relation-extraction-on-3)](https://paperswithcode.com/sota/joint-entity-and-relation-extraction-on-3?p=rebel-relation-extraction-by-end-to-end)
[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/rebel-relation-extraction-by-end-to-end/relation-extraction-on-ade-corpus)](https://paperswithcode.com/sota/relation-extraction-on-ade-corpus?p=rebel-relation-extraction-by-end-to-end)
[![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/rebel-relation-extraction-by-end-to-end/relation-extraction-on-re-tacred)](https://paperswithcode.com/sota/relation-extraction-on-re-tacred?p=rebel-relation-extraction-by-end-to-end)

[![Hugging Face Models](https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-REBEL-blue)](https://huggingface.co/Babelscape/rebel-large)
[![Hugging Face Spaces](https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Spaces-blue)](https://huggingface.co/spaces/Babelscape/rebel-demo)
[![](https://img.shields.io/badge/-PyTorch--Lightning--Template-blueviolet?style=for-the-badge&logo=github)](https://github.com/edobobo/p-lightning-template)

# REBEL: Relation Extraction By End-to-end Language generation

This is the repository for the Findings of EMNLP 2021 paper REBEL: Relation Extraction By End-to-end Language generation. We present a new linearization aproach and a reframing of Relation Extraction as a seq2seq task. The paper can be found [here](docs/EMNLP_2021_REBEL__Camera_Ready_.pdf). If you use the code, please reference this work in your paper:

    @inproceedings{huguet-cabot-navigli-2021-rebel-relation,
        title = "{REBEL}: Relation Extraction By End-to-end Language generation",
        author = "Huguet Cabot, Pere-Llu{\'\i}s  and
          Navigli, Roberto",
        booktitle = "Findings of the Association for Computational Linguistics: EMNLP 2021",
        month = nov,
        year = "2021",
        address = "Punta Cana, Dominican Republic",
        publisher = "Association for Computational Linguistics",
        url = "https://aclanthology.org/2021.findings-emnlp.204",
        pages = "2370--2381",
        abstract = "Extracting relation triplets from raw text is a crucial task in Information Extraction, enabling multiple applications such as populating or validating knowledge bases, factchecking, and other downstream tasks. However, it usually involves multiple-step pipelines that propagate errors or are limited to a small number of relation types. To overcome these issues, we propose the use of autoregressive seq2seq models. Such models have previously been shown to perform well not only in language generation, but also in NLU tasks such as Entity Linking, thanks to their framing as seq2seq tasks. In this paper, we show how Relation Extraction can be simplified by expressing triplets as a sequence of text and we present REBEL, a seq2seq model based on BART that performs end-to-end relation extraction for more than 200 different relation types. We show our model{'}s flexibility by fine-tuning it on an array of Relation Extraction and Relation Classification benchmarks, with it attaining state-of-the-art performance in most of them.",
    }


```
Repo structure
| conf  # contains Hydra config files
  | data
  | model
  | train
  root.yaml  # hydra root config file
| data  # data
| datasets  # datasets scripts
| model # model files should be stored here
| src
  | pl_data_modules.py  # LightinigDataModule
  | pl_modules.py  # LightningModule
  | train.py  # main script for training the network
  | test.py  # main script for training the network
| README.md
| requirements.txt
| demo.py # Streamlit demo to try out the model
| setup.sh # environment setup script 
```

## Initialize environment
In order to set up the python interpreter we utilize [conda](https://docs.conda.io/projects/conda/en/latest/index.html)
, the script setup.sh creates a conda environment and install pytorch
and the dependencies in "requirements.txt". 

## REBEL Model and Dataset

Model and Dataset files can be downloaded here:

https://osf.io/4x3r9/?view_only=87e7af84c0564bd1b3eadff23e4b7e54

Or you can directly use the model from Huggingface repo:

https://huggingface.co/Babelscape/rebel-large

```python
from transformers import pipeline

triplet_extractor = pipeline('text2text-generation', model='Babelscape/rebel-large', tokenizer='Babelscape/rebel-large')

# We need to use the tokenizer manually since we need special tokens.
extracted_text = triplet_extractor.tokenizer.batch_decode(triplet_extractor("Punta Cana is a resort town in the municipality of Higuey, in La Altagracia Province, the eastern most province of the Dominican Republic", return_tensors=True, return_text=False)[0]["generated_token_ids"]["output_ids"])

print(extracted_text[0])

# Function to parse the generated text and extract the triplets
def extract_triplets(text):
    triplets = []
    relation, subject, relation, object_ = '', '', '', ''
    text = text.strip()
    current = 'x'
    for token in text.replace("<s>", "").replace("<pad>", "").replace("</s>", "").split():
        if token == "<triplet>":
            current = 't'
            if relation != '':
                triplets.append({'head': subject.strip(), 'type': relation.strip(),'tail': object_.strip()})
                relation = ''
            subject = ''
        elif token == "<subj>":
            current = 's'
            if relation != '':
                triplets.append({'head': subject.strip(), 'type': relation.strip(),'tail': object_.strip()})
            object_ = ''
        elif token == "<obj>":
            current = 'o'
            relation = ''
        else:
            if current == 't':
                subject += ' ' + token
            elif current == 's':
                object_ += ' ' + token
            elif current == 'o':
                relation += ' ' + token
    if subject != '' and relation != '' and object_ != '':
        triplets.append({'head': subject.strip(), 'type': relation.strip(),'tail': object_.strip()})
    return triplets
extracted_triplets = extract_triplets(extracted_text[0])
print(extracted_triplets)
```

### CROCODILE: automatiC RelatiOn extraCtiOn Dataset wIth nLi filtEring.

REBEL dataset can be recreated using our RE dataset creator [CROCODILE](https://github.com/Babelscape/crocodile)

## Training and testing

There are conf files to train and test each model. Within the src folder to train for CONLL04 for instance:

    train.py model=rebel_model data=conll04_data train=conll04_train

Once the model is trained, the checkpoint can be evaluated by running:

    test.py model=rebel_model data=conll04_data train=conll04_train do_predict=True checkpoint_path="path_to_checkpoint"

src/model_saving.py can be used to convert a pytorch lightning checkpoint into the hf transformers format for model and tokenizer.


## DEMO

We suggest running the demo to test REBEL. Once the model files are unzipped in the model folder run:

    streamlit run demo.py

And a demo will be available in the browser. It accepts free input as well as data from the sample file in data/rebel/

## Datasets

TACRED is not freely avialable but instructions on how to create Re-TACRED from it can be found [here](https://github.com/gstoica27/Re-TACRED).

For CONLL04 and ADE one can use the script from the [SpERT github](https://github.com/lavis-nlp/spert/blob/master/scripts/fetch_datasets.sh).

For NYT the dataset can be downloaded from [Copy_RE github](https://github.com/xiangrongzeng/copy_re).

Finally the DocRED for RE can be downloaded at the [JEREX github](https://github.com/lavis-nlp/jerex/blob/main/scripts/fetch_datasets.sh)

# Tagger and Parser

## Environment setup
1. Create a conda environment with Python 3.7
```
conda create -n allennlp python=3.7
```
2. Activate the new environment
```
conda activate allennlp
```
3. Install allennlp (we use version 0.8.4) and other packages using pip 
```
pip install -r requirements.txt
```

**Internal note**: both environments are already set up on coli servers, see instructions in the [Wiki](https://github.com/interactive-cookbook/tagger-parser/wiki/Setup-on-the-coli-servers).

## Parameter configuration

Adjust parameters including file paths in the respective `.json` config files, as needed. By default, the paths point to datasets in [`data`](data). See respective README files there for details about the datasets. 

Both our models consume data in CoNLL format where each line represents a token and columns are tab-separated. The column DEPRELS contains additional dependency relations if a token has more than one head.The tagger requires data in the [CoNLL-2003](https://www.clips.uantwerpen.be/conll2003/ner/) format with the relevant columns being the first (TEXT) and the fourth (LABEL). The parser requires data in the [CoNLL-U](https://universaldependencies.org/format.html) format with the relevant columns being the second (FORM), the  fifth (LABEL), the seventh (HEAD) and the eighth (DEPREL). 

Available AllenNLP 0.8 configurations:
- `tagger/tagger_with_bert_config.json` - BiLSTM-CNN-CRF tagger using BERT embeddings
- `tagger/tagger_with_english_elmo_config.json` - BiLSTM-CNN-CRF tagger using English ELMo embeddings
- `tagger/tagger_with_german_elmo_config.json` - BiLSTM-CNN-CRF tagger using German ELMo embeddings
- `parser/parser_config.json` - Biaffine dependency parser (Dozat and Manning, 2017)

For the ELMo taggers, we use the following ELMo parameters (i.e. options and weights):
- [English](https://api.semanticscholar.org/CorpusID:7197241): [weights and options](https://allennlp.s3.amazonaws.com/models/ner-model-2018.12.18.tar.gz) (use the weights and options files under `fta/` after unzipping)
- [German](https://github.com/t-systems-on-site-services-gmbh/german-elmo-model): [weights](https://github.com/t-systems-on-site-services-gmbh/german-elmo-model/releases/download/files_1/weights.hdf5) and [options](https://github.com/t-systems-on-site-services-gmbh/german-elmo-model/releases/download/files_1/options.json)

**Internal note**: the ELMo options and weight files can be found on the Saarland servers at `/proj/cookbook/`.

The weights and options files should be named and placed according to the paths specified in the .json files; alternatively, adjust the paths in the .json files.

## Training

Run `allennlp train [params] -s [serialization dir]` to train a model, where
- `[params]` is the path to the .json config file.
- `[serialization dir]` is the directory to save trained model, logs and other results.

## Evaluation
Run `allennlp evaluate [archive file] [input file] --output-file [output file]` to evaluate the model on some evaluation data, where
- `[archive file]` is the path to an archived trained model.
- `[input file]` is the path to the file containing the evaluation data.
- `[output file]` is an optional path to save the metrics as JSON; if not provided, the output will be displayed on the console.

### Performance

**ERRATUM** (Donatelli et al., EMNLP 2021): Please refer to our [Wiki page](https://github.com/interactive-cookbook/tagger-parser/wiki/Erratum-(EMNLP-2021-Paper)) for a list of corrections, particularly concerning the reporting of results and comparability.

<!-- 
Tagger performance on the [English corpus](data/English) (test.conll03):

Embedder | Precision | Recall | F-Score
--- | --- | --- | ---
English ELMo | 89.9 | 89.2 | 89.6 
multilingual BERT | 88.7 | 88.4 | 88.5 
-->
Our tagger's performance compared to Y'20's performance and inter-annotator agreement (IAA).

Model | Corpus | Embedder | Precision  | Recall | F-Score  
--- | --- | --- | --- | --- | ---
IAA | 100-r by Y'20 | | 89.9 | 92.2 | 90.5
| | | | | 
Y'20 | 300-r by Y'20 | | 86.5 | 88.8 | 87.6
Our tagger  | [300-r by Y'20](data/English/Tagger) | [English ELMo](tagger/tagger_with_english_elmo_config.json) | **89.9** ± 0.5 | **89.2** ± 0.4 | **89.6** ± 0.3
Our tagger  | [300-r by Y'20](data/English/Tagger) | [multilingual BERT](tagger/tagger_with_bert_config.json) | 88.7 ± 0.4 | 88.4 ± 0.1 | 88.5 ± 0.2
| | | | | 
Our tagger  | [German](data/German/Tagger) | [German ELMo](tagger/tagger_with_german_elmo_config.json) | 79.2 ± 1.4 | 81.2 ± 1.8 | 80.2 ± 1.6
Our tagger  | [German](data/German/Tagger) | [multilingual BERT](tagger/tagger_with_bert_config.json) | 75.3 ± 0.8 | 76.0 ± 1.0 | 75.7 ± 0.9



<!-- 
Parser performance on the [English corpus](https://github.com/interactive-cookbook/tagger-parser/tree/main/data/English/Parser) (test.conllu):

Tag Source | Precision | Recall | F-Score
--- | --- | --- | ---
gold tags | 80.4 | 76.1 | 78.2 
our tagger with ELMo embeddings | 74.4 | 70.4 | 72.3
-->

Our parser's performance compared to Y'20's performance and inter-annotator agreement (IAA).

Model | Corpus |  Tag source | Precision  | Recall | F-Score 
--- | --- | --- | --- | --- | --- 
IAA | 100-r by Y'20 | gold tags | 84.4 | 80.4 | 82.3
 |  |  |  |  |
Y'20 | 300-r by Y'20 | gold tags | 73.7 | 68.6 | 71.1
Our parser | [300-r by Y'20](data/English/Parser) | gold tags | 80.4 ± 0.0 | 76.1 ± 0.0 | **78.2** ± 0.0
 |  |  |  |  |
Our parser  | [German](data/German/Parser) | gold tags | 69.3 ± 0.0 | 91.3 ± 0.0 | 78.8 ± 0.0

Our parser's performance on machine-tagged data:

Model | Corpus |  Tag source | Precision  | Recall | F-Score 
--- | --- | --- | --- | --- | --- 
Y'20 | 300-r by Y'20 | Y'20 tagger | 51.1 | 37.7 | 43.3
Our parser  | [300-r by Y'20](data/English/Parser) | [our ELMo tagger](tagger/tagger_with_english_elmo_config.json) | 74.4 ± 0.5 | 70.4 ± 1.0 | **72.3** ± 0.8
Our parser  | [German](data/German/Parser) | [German ELMo tagger](tagger/tagger_with_german_elmo_config.json) | 56.5 ± 1.1 | 82.8 ± 2.2 | 67.1 ± 0.5

## Prediction

Run `allennlp predict [archive file] [input file] --use-dataset-reader --output-file [output file]` to parse a file with a pretrained model, where
- `[archive file]` is the path to an archived trained model.
- `[input file]` is the path to the file you want to parse; this file should be in the same format as the training data, i.e. CoNLL-2003 for the tagger and CoNLL-U for the parser.
- `use-dataset-reader` tells the parser to use the same dataset reader as it used during training.
- `[output file]` is an optional path to save parsing results as JSON; if not provided, the output will be displayed on the console.

The output of the parser will be in JSON format. To transform this into the better readable CoNLL-U format, use [data-scripts/json_to_conll.py](data-scripts/json_to_conll.py). To get labeled evaluation results for parser output, use the script [data-scripts/parser_evaluation.py]([data-scripts/parser_evaluation.py]). Instructions for their use can be found in [data-scripts/README.md](data-scripts/README.md).

For sample inputs and outputs see [English/Samples](data/English/Samples). 

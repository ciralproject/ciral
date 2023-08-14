# 🌍 CIRAL

CIRAL (Cross-Lingual Information Retrieval for African Languages) is a track at the Forum for Information Retrieval Evaluation (FIRE) 2023, focused on promoting the research and evaluation of Cross-Lingual Information Retrieval (CLIR) for solely African languages. A goal of the track is to develop the first human-annotated ad-hoc CLIR test collections for African languages, starting off with 4 languages and subsequent addition of others. This repo provides details of the test collection and guidelines for the task.

More information regarding the track can be found at the website: [Ciral@Fire2023](https://ciralproject.github.io/)



## 📚 Corpora

The focus languages in this first implementation of the task are Hausa, Swahili, Somali and Yoruba. The corpora consists of passages from news articles, mined from indigenous websites of the different languages. 

Link to Dataset: https://huggingface.co/datasets/CIRAL/ciral-corpus/

Statistics and details of the collection are found below.

| Language        | News Sources                           | # of Passages | # of Articles | Link |
|:-------|:------------|:-------:|:-------:|:----:|
| Hausa (hau)     | LegitNG, DailyTrust, VOA, Isyaku, etc. |       715,355 |       240,883 | [🤗](https://huggingface.co/datasets/CIRAL/ciral-corpus/tree/main/passages-v1.0)
| Swahili (swa)   | VOA, Tuko, Risaala, Caasimada, etc.    |       981,658 |       146,669 | [🤗](https://huggingface.co/datasets/CIRAL/ciral-corpus/tree/main/passages-v1.0)
| Somali (som)    | VOA, UN Swahili, MTanzania, etc.       |     1,015,567 |       629,441 | [🤗](https://huggingface.co/datasets/CIRAL/ciral-corpus/tree/main/passages-v1.0)
| Yoruba (yor)    | Alaroye, VON, BBC, Asejere, etc.       |        82,095 |        27,985 | [🤗](https://huggingface.co/datasets/CIRAL/ciral-corpus/tree/main/passages-v1.0)


For each language, passages are stored in [JSONL](https://jsonlines.org/) files where each line corresponds to a passage in JSON format. The fields provided for each passage include:
-  `docid`: Unique identifier of the passage
- `title`: Title of the news article from which the passage was extracted
- `text`: Text of the passage
- `url`: News article url


## 📚 Queries and Relevance Judgements

The CIRAL queries and relevance judgements are provided for each of the four languages in two sets: train and test sets. They can be accessed in the [Hugging Face repo](https://huggingface.co/datasets/CIRAL/ciral)

Statistics for the Train queries and relevance judgements are given below. The train set consists of few samples to analyze relevance and evaluate proposed systems using the provided judgements. 

Statistics for the collection would be completed after **pooling**.

| Language  | # Train Queries | # Train Judgements | Link |
|:-------|:---:|:-------:|:------:|
| Hausa (ha)  | 10 | 165 | [🤗](https://huggingface.co/datasets/CIRAL/ciral/tree/main/ciral-hausa)| 
| Swahili (sw)   | 10 | 196 | [🤗](https://huggingface.co/datasets/CIRAL/ciral/tree/main/ciral-swahili) | 
| Somali (so)    | 10 | 187 | [🤗](https://huggingface.co/datasets/CIRAL/ciral/tree/main/ciral-somali) | 
| Yoruba (yo)    | 10 | 185 | [🤗](https://huggingface.co/datasets/CIRAL/ciral/tree/main/ciral-yoruba) |

Both query and relevance judgements files are in the `.tsv` format. Each line in the query file is represented as:

```
qid\tquery
```

while the judgements are in the standard TREC format:

```
qid Q0 docid relevance
```


## 📚 Guidelines and Resources

#### Task Description
This task entails queries formulated as natural language questions in English, and retrieval done at the passage-level for the different African languages. Information retrieval systems developed for the task will receive the collection of passages and a set of queries for the different African languages. For each query in the test set, proposed systems are to return a ranked list of passages ordered by likelihood of binary relevance to the query. Up to 1000 passages per query can be submitted, results with more than 1000 would be truncated.

Details regarding participation can be found in this [section](https://ciralproject.github.io/#participation) of the website.


#### Getting started with IR and CIRAL
For more details on getting started with IR and understanding the task, please check the provided [Quick Start](Guidelines/README.md)



## 🔎 Baseline and Evaluation

Baselines and reproduction guides are provided in this section. Please note that this only covers searching, as the indexes have already been built. <!--Add links to reproduce indexes.-->

The baseline can be reproduced using [Pyserini](https://github.com/castorini/pyserini). This would require:
- Installing the development version of Pyserini according to these [guide](https://github.com/castorini/pyserini/blob/master/docs/installation.md#development-installation).
- Coping the topic and qrel files from the [Hugging Face repo](https://huggingface.co/datasets/CIRAL/ciral) to `tools/topics-and-qrels` in the cloned `Pyserini` repo. 

```bash
git clone https://huggingface.co/datasets/CIRAL/ciral
cp -r ciral/*/*/* $PYSERINI_PATH/tools/topics-and-qrels/
```

### Afriberta-DPR

#### Batch retrieval run on the Afriberta-DPR baseline:

```bash
lang=yo # or ha, so, sw
set=train # or test

tools_dir=tools/topics-and-qrels
run_file=runs/run.ciral.afriberta-dpr.${lang}.${set}.txt

python -m pyserini.search.faiss \
  --encoder-class auto \
  --encoder castorini/afriberta-dpr-ptf-msmarco-ft-latin-mrtydi  \
  --topics ${tools_dir}/topics.ciral-v1.0-${lang}-${set}.tsv \
  --index $INDEX_DIR \
  --output ${run_file} --batch 128 --threads 16 --hits 1000
```

#### Evaluation

```bash
python -m pyserini.eval.trec_eval \
  -c -M 100 -m ndcg_cut.20 -m recall.100 ${tools_dir}/qrels.ciral-v1.0-${lang}-${set}.tsv ${run_file}
```


## Results: Train Set

We present the ncdg@20 and recall@100 scores for the baselines.

#### NDCG@20
|                |   Hausa  |   Somali  |   Swahili  |   Yoruba  |
|----------------|:--------:|:---------:|:----------:|:---------:|
| Afriberta-DPR  | 0.2950   |   0.1509  |   0.3130   |   0.0638  |



#### Recall@100
|                |   Hausa  |   Somali  |   Swahili  |   Yoruba  |
|----------------|:--------:|:---------:|:----------:|:---------:|
| Afriberta-DPR  |  0.6035  |   0.3044  |   0.5463   |   0.1652  |





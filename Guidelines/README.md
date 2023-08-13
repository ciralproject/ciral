## Quick Start on Information Retrieval (IR) and the CIRAL Task

This guide is for any one who would love to participate and make a submission to the CIRAL track, with or without an information retrieval background. CIRAL focuses on cross-lingual (from one language to a different language) information retrieval for African languages, with the goal of carrying out community evaluations and in general building community.

For a quick overview of the task, please refer to the main [README](../README.md) of the repo or the track's [website](https://ciralproject.github.io/).

<br />

## 📚 Introduction to IR
As a great start into IR, we would majorly be working with two toolkits designed to foster research in information retrieval. First is [Anserini](https://github.com/castorini/anserini), built in Java and on the [Lucene search library](https://lucene.apache.org/) and then [Pyserini](https://github.com/castorini/pyserini) which works for both sparse and dense representations and has Anserini integrated. The list provided is a good path to follow sequentially in getting started with the basics of IR (each next documents is also linked in the previous one).

- To understand the retrieval problem, high-level of retrieval systems and core concepts - [Anserini: Start Here](https://github.com/castorini/anserini/blob/master/docs/start-here.md)

- Using Anserini to Index, Search and Evaluate - [Anserini: BM25 Baselines for MS MARCO Passage Ranking](https://github.com/castorini/anserini/blob/master/docs/experiments-msmarco-passage.md)

- Using Pyserini to Index, Search and Evaluate - [Pyserini: BM25 Baseline for MS MARCO Passage Ranking](https://github.com/castorini/pyserini/blob/master/docs/experiments-msmarco-passage.md)

- Learn about the relationship between sparse and dense retrieval. [Pyserini: A Conceptual Framework for Retrieval](https://github.com/castorini/pyserini/blob/master/docs/conceptual-framework.md)

- Working with an Actual Dense Retrieval model. [Pyserini: Contriever Baseline for NFCorpus](https://github.com/castorini/pyserini/blob/master/docs/experiments-nfcorpus.md)

<br />


## 📚 Batch Retrieval with the CIRAL Train Queries
Now that the basic understanding of IR, Anserini and Pyserini has been accomplished, we can try out some very simple retrieval with the provided `train` queries in CIRAL using BM25. This would be done using [Pyserini](https://github.com/castorini/pyserini):

1. If not done already, clone Pyserini and install the development version of Pyserini according to these [guide](https://github.com/castorini/pyserini/blob/master/docs/installation.md#development-installation). 

2. Using the following commands, copy the topic and qrel files from the [Hugging Face repo](https://huggingface.co/datasets/CIRAL/ciral) to `tools/topics-and-qrels` in the cloned `Pyserini` repo. 

```bash
git clone https://huggingface.co/datasets/CIRAL/ciral
cp -r ciral/*/*/* $PYSERINI_PATH/tools/topics-and-qrels/
```

3. Run batch retrieval using CIRAL's [pre-built BM25 indexes](https://github.com/castorini/pyserini/blob/master/docs/prebuilt-indexes.md). `{lang}` represents language code for any of the four languages: yo (Yoruba), so (Somali), ha (Hausa) or sw (Swahili). 

```bash
python -m pyserini.search.lucene \
  --language {lang} \
  --topics tools/topics-and-qrels/topics.ciral-v1.0-{lang}-train.tsv \
  --index ciral-v1.0-{lang} \
  --output runs/run.ciral-v1.0-{lang}.bm25.train.txt \
  --pretokenized \
  --batch 128 --threads 16 --bm25 --hits 1000
```

This saves the run (retrieved passages) in `runs/run.ciral-v1.0-{lang}.bm25.train.txt`. You can inspect the file to see what the output (in other words submission files) looks like.

4. Next, we  evaluate the run. The official metrics for the track are `ndcg@20` and `recall@100`, but we only evaluate for `recall@1000` in this case (i.e the number of correct passages returned in all the 1000 passages per query)

```bash
python -m pyserini.eval.trec_eval -c -m recall.1000 tools/topics-and-qrels/qrels.ciral-v1.0-{lang}-train.tsv runs/run.ciral-v1.0-{lang}.bm25.train.txt
```

This should give the following results:
| **recall@1000** | |  
|:----------------|:---:|
|**Language** | **BM25 (default)** |
| Yoruba (ha)     | 0.6010 |
| Swahili (sw)     | 0.1333 |
| Somali (ha)     | 0.1267 |
| Hausa (ha)     | 0.1050 |

</br>

### Dense Retrieval 
Already built dense indexes have been provided and can be downloaded [here](). For a recap on how to build dense indexes, you can also take a look (https://github.com/castorini/pyserini/blob/master/docs/experiments-nfcorpus.md) or more generally [here](https://github.com/castorini/pyserini/blob/master/docs/usage-index.md#building-a-dense-vector-index).

</br>

## 📚 Training Dense Retrieval Models
To train or finetune your own dense retrieval model, the [Tevatron](https://github.com/texttron/tevatron/tree/main/src/tevatron) toolkit is a good place to start. 
 - [Examples](https://github.com/texttron/tevatron/tree/main/examples) on different retrieval tasks.
- [Documentation](http://tevatron.ai/)

</br>

## 📚 Additional Data sources

CIRAL is a test collection, hence the queries and qrels provided would not train a dense retriever efficiently. Therefore, we suggest more multilingual and cross-lingual datasets in training/finetuning:

- AfriCLIRMatrix: Multilingual CLIR collection for African languages, with topics in English and documents in African languages. Contains data for all four languages. [Dataset](https://huggingface.co/datasets/castorini/africlirmatrix)

- MIRACL: A multilingual dataset, with queries formulated as natural language questions, and retrieval done at the passage level. Includes Swahili and Yoruba. [Dataset](https://huggingface.co/datasets/miracl/miracl).

- CLIRMatrix: Another multilingual CLIR collection which includes Swahili and Yoruba. [Dataset](https://ir-datasets.com/clirmatrix.html)

- WikiCLIR: Includes only Swahili.
[Dataset](https://ir-datasets.com/master/wikiclir.html),
[Swahili](https://ir-datasets.com/master/wikiclir.html#wikiclir/sw)


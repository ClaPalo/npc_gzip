# A Replication Study of *“Low-Resource” Text Classification: A Parameter-Free Classification Method with Compressors*

Original paper was accepted to Findings of [ACL2023](https://aclanthology.org/2023.findings-acl.426/).

Original GitHub repository: https://github.com/bazingagin/npc_gzip

## Getting Started

This codebase is also [available on pypi.org via](https://pypi.org/project/npc-gzip):

```sh
pip install npc-gzip
```

## Usage

See the [examples](./examples/) directory for some examples.

## Testing

This package utilizes `poetry` to maintain its dependencies and `pytest` to execute tests. To get started running the tests:

```sh
poetry shell
poetry install
pytest
```

WARNING: According to [the official documentation](https://python-poetry.org/docs/), *Poetry requires Python 3.8+*. We found out that you have to install Python 3.9+ to be able to run both Poetry and the original codebase. Take this into account if you want to execute tests.

---

### Original Codebase

#### Require

See `requirements.txt`.

Install requirements in a clean environment:

```sh
conda create -n npc python=3.7
conda activate npc
pip install -r requirements.txt
```

#### Run

```sh
python main_text.py
```

By default, this will only use 100 test and training samples per class as a quick demo. They can be changed by `--num_test`, `--num_train`. The command should also create a new directory, called `data`, containing the `train.csv` and `test.csv` files.

```text
--compressor <gzip, lzma, bz2>
--dataset <AG_NEWS, SogouNews, DBpedia, YahooAnswers, 20News, Ohsumed_single, R8, R52, kinnews, kirnews, swahili, filipino>
--num_train <INT>
--num_test <INT>
--data_dir <DIR>
--all_test
--all_train
--record [This will record the distance matrix in order to save it for future uses. It's helpful when you want to run on the whole dataset.]
--test_idx_start <INT>
--test_idx_end <INT> [These two args help us to run on a certain range of test set. Also helpful for calculating the distance matrix on the whole dataset.]
--para [This will use multiprocessing to accelerate.]
--output_dir <DIR> [The output directory to save information of tested indices or distance matrix.]
```

WARNING: We found out that the program is not always able to automatically download the dataset .csv files required to run the text-classification algorithm. The original authors didn't specify it neither in the repo nor in the paper and it took us a lot of time to understand this. If you experience errors like *FileNotFoundError: [Errno 2] No such file or directory: 'data/train.csv'* or *NotImplementedError: Loading a dataset cached in a LocalFileSystem is not supported.* you will probably have to create a directory in the `data/` path on your own. Then you will have to specify the directory location using the `--data_dir` argument. The directory must store at least a train.csv file and a test.csv file. Whenever the program didn't automatically load the files, we tried to download these files for you from the Internet and accordingly enrich the `data` directory in this repository. However, at the moment, we didn't manage to run experiments on the R8, R52, Ohsumed and SogouNews files. It seems like the program is not able to correctly fetch them. Probably, either the files we found have a different format than the one used by the original authors or data.py contains some bugs to be fixed.
As a general guideline, unless you want to use `YahooAnswers`, `DBpedia` or `20News`, we suggest you to try to use the files we have put in `data/` to avoid triggering the *NotImplementedError: Loading a dataset cached in a LocalFileSystem is not supported.* If you still experience problems, like *TypeError: MapperIterDataPipe instance doesn't have valid length*, try to use different files in `data/` for the dataset you are insterested in.

EDIT: Unfortunately the sogou_news dataset is too large to be ulpoaded on GitHub. You can download it on your own from [here](https://s3.amazonaws.com/fast-ai-nlp/sogou_news_csv.tgz)

#### Run full-shot experiments

According to the authors, you should be able to run experiments on entire datasets (i.e. full-shot) and take advantage of multiprocessing by using:

```sh
python main_text.py --para --dataset <DATASET> --all_train --all_test
```

However, with large datasets, such as AG_NEWS, DBpedia and YahooAnswers, we received errors saying that the automatically set amount of train and test samples was not acceptable (e.g. *ValueError: Cannot take a larger sample than population when 'replace=False'*). To fix this, we ran experiments to ensure a full-shot scenario (by finding the highest possible `num_train` and `num_test` through fine-tuning). In the case of AG-NEWS, they were 30000 and 1900, respectively:

```sh
python main_text.py --para --dataset AG_NEWS --num_train 30000 --num_test 1900
```

WARNING: Given the authors' claims about how the npc method should be lighter and since they did not explicitly specify what kind of machine they used to obtain the results, we expected that a C2-standard-8 machine on Google Cloud Platform would be sufficient to obtain results in a reasonable time. However, we found that full experiments on some of the large datasets mentioned above can require long CPU times. To give an idea:
the full-shot experiment on DBPedia took 6.42 hours, the one on AG_NEWS 3.7894 hours. Running a full-shot experiment on YahooAnswers turned out to be impractical because the execution was interrupted before it even started. To get at least an approximate result even on this dataset, we ran an experiment with 14000 samples (out of 140000, corresponding to full-shot) that took 0.904339 hours.

#### Run 5-shot experiments

To run 5-shot experiments, use:

```sh
python main_text.py --para --dataset <DATASET> --num_train 5 --all_test
```

#### Calculate Accuracy (Optional)

If we want to calculate accuracy from recorded distance file `<DISTANCE DIR>`, use

```sh
python main_text.py --record --score --distance_fn <DISTANCE DIR>
```

Anyway, the accuracy will always be automatically calculated regardless of the arguments you specify.

#### Use Custom Dataset (We didn't try this)

You can use your own custom dataset by passing `custom` to `--dataset`; pass the data directory that contains `train.txt` and `test.txt` to `--data_dir`; pass the class number to the `--class_num`.

Both `train.txt` and `test.txt` are expected to have the format `{label}\t{text}` per line.

You can change the delimiter according to you dataset by changing `delimiter` in `load_custom_dataset()` in `data.py`.

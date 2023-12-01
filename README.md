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
--all_test [This will use the whole test dataset.]
--all_train
--record [This will record the distance matrix in order to save it for future uses. It's helpful when you want to run on the whole dataset.]
--test_idx_start <INT>
--test_idx_end <INT> [These two args help us to run on a certain range of test set. Also helpful for calculating the distance matrix on the whole dataset.]
--para [This will use multiprocessing to accelerate.]
--output_dir <DIR> [The output directory to save information of tested indices or distance matrix.]
```

WARNING: We found out that the program is not always able to automatically download the dataset .csv files required to run the text-classification algorithm. The original authors didn't specify this neither in the repo nor in the paper and it took us a lot of time to understand this. If you experience errors like *FileNotFoundError: [Errno 2] No such file or directory: 'data/train.csv'* or *NotImplementedError: Loading a dataset cached in a LocalFileSystem is not supported.* you will probably have to create a directory in the `data/` path on your own. Then you will have to specify the directory location using the `--data_dir` argument. The directory must store at least a train.csv file and a test.csv file. Both the files must have a `{label}\t{text}` format. When possible, we downloaded these files for you and we accordingly enriched the `data` directory in this repository. However, we didn't manage to easily find on the Internet the files necessary to run the classification on R8, R52 and ohsumed.

#### Run full-shot experiments

To run full-shot experiments, use:

```sh
python main_text.py --para --dataset <DATASET> --all_train --all_test
```

With certain datasets, such as DBpedia, we were obtaining an error while doing so. To fix this, we ran the experiment with the highest num_train possible (found through tuning). In the case of DBpedia, it was 40000:

```sh
python main_text.py --para --dataset DBpedia --num_train 40000 --all_test
```

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

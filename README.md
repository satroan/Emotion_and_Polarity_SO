# EmoTxt
A toolkit for emotion detection from technical text

## Fair Use Policy
Please, cite the following paper if you intend to use our tool for your own research:
> F. Calefato, F. Lanubile, N. Novielli. “[EmoTxt: A Toolkit for Emotion Recognition from Text](https://arxiv.org/abs/1708.03892)” To appear in *Proceedings of ACII 2017, the 7th International Conference on Affective Computing and Intelligent Interaction*, San Antonio, USA, Oct. 23-26, 2017.

## Requirements
* Ram: 16GB or more
* Python 2.7.x
  * Libraries
    * `nltk-3.2.2`, `numpy-1.11.3+mkl-cp27`, `scikit_learn-0.15.1-cp27`, `scipy-0.18.1-cp27`, `pattern-2.6`
      * Installation: open the command line and run
      
        `$ pip install -r requirements.txt`
    * Stanford CoreNLP models
      * Installation: download it from [here](http://nlp.stanford.edu/software/stanford-corenlp-models-current.jar), then move the jar file to the `./java/lib/` subfolder.
* Java 8+
* R
  * Libraries:
    * `caret`, `LiblinearR` , `e1071`
      * Installation: open the command line and run
      
        `$ Rscript requirements.R`

## Usage
In the following, we show first how to train a new model for emotion classification and, then, how to test the model on unseen data.

### Training a new model for emotion classification (70/30% split for train and test)
```
$ train.sh -i file.csv -d delimiter [-g] -e emotion 
```

where:

* `-i file.csv`: the input file coded in **UTF-8 without BOM**, containing the input corpus; the format of the input file is specified [here](https://github.com/collab-uniba/Emotion_and_Polarity_SO/wiki/File-format-for-training-corpus). Please, not that gold label are required for each item in the dataset
* `-d delimiter`: the delimiter used in the csv file (values in {`c`, `sc`}, where stands for comma and sc for semicolon)
* `-g`: enables the extraction of n-grams (i.e,. bigrams and unigrams). N-grams extraction is mandatory for the first run when you want to train a new classification model for a given emotion, using your own dataset for the first time. Because n-gram extraction is computationally expensive, it should be skipped if you retrain the model for the same emotion using the same input file.
* `-e emotion`: the specific emotion for which you want to train a classification model, with values in {`joy`, `anger`, `sadness`, `love`, `surprise`, `fear`}

As a result, the script will generate the following output files:

* An outputn folder named `training_<file.csv>_<emotion>/`, containing:
   * `n-grams/`: a subfolder containing the extracted n-grams
   * `idfs/`: a subfolder containing the IDFs computed for n-grams and WordNet Affect emotion words
   * `feature-<emotion>.csv`: a .csv file with the features extracted from the input corpus and used for training the model
   * `liblinear/`:
     * there are two subfolders: `DownSampling/` and `NoDownSampling/`. Each one contains:
          * `trainingSet.csv`
          * `testingSet.csv`
          * eight models trained with liblinear `model_<emotion>_<IDMODEL>.Rda`, where `IDMODEL` is the ID of the liblinear model, with values in `{0,...,7}`):
          * `performance_<emotion>_<IDMODEL>.txt`, containing the results of the parameter tuning for the model (best C) as performed by caret, the confusion matrix and the precision, recall and f-measure for the best cost for the specific emotion
          * `predictions_<emotion>_<IDMODEL>.csv`, containing the test instances with the predicted labels for the specific emotion

### Emotion detection
```
$ classify.sh -i file.csv -d delimiter -e emotion [-m model] [-f idf] [-o n-grams] [-l]
```

where:

* `-i file.csv`: the input csv file with header and coded in **UTF-8 without BOM**, containing the corpus to be classified; the format of the input file is the following: 

  ```
  id;label;text
  ...
  22;NO;"""Excellent! This is exactly what I needed. Thanks!"""
  23;YES;"""FEAR!!!!!!!!!!!"""
  ...
  ```
* `-d delimiter`: the delimiter used in the csv file (values in {`c`, `sc`}, where stands for comma and sc for semicolon)
* `-e emotion`: the specific emotion to be detected in the input file or text, defined in {`joy`, `anger`, `sadness`, `love`, `surprise`, `fear`}
* `-m model`: the model file learnt during the training step (e.g., `model-anger.rda`). If you don't specify the model name, the default model will be used, that is the one learnt on our Stack Overflow gold standard
* `-o n-grams`: if you specify a model name using `-m` (i.e., you don't want to use the default model for a given emotion) you are required to provide also the path to the folder containing the dictionaries extracted during the training step. This folder includes n-grams, i.e., `UnigramsList.txt` and `BigramsList.txt`. 
* `-f idf`: if you specify a model name using `-m` (i.e., you don't want to use the default model for a given emotion) you are required to specify also the path to the folder containing the dictionaries with IDFs computed during the training step. The folder includes IDFs for n-grams (uni- and bi-grams) and for WordNet Affect lists of emotion words.
* `-l`: if presents , indicates  `<file.csv>` contains a gold label in the column `label`.

As a result, the script will create an output folder named `classification_<file.csv>_<emotion>` containing:

* `predictions_<emotion>.csv`: a csv file with header, containing a binary prediction (yes/no) for each line of the input corpus:

  ```
  id;predicted
  ...
  22;NO
  23;YES
  ...
  ```
* `performance_<emotion>.txt`: a file created only if the input corpus `<file.csv>` contains the column `label`; the file contains several performance metrics (Precision, Recall, F1, confusion matrix).

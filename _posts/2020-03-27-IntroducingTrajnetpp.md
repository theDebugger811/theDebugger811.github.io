---
title: 'Introducing TrajNet++ : A Framework for Human Trajectory Forecasting'
date: 2020-03-27
permalink: /posts/2020/03/intro_trajnetpp/
tags:
  - trajectory prediction
  - autonomous systems
  - social intelligence
---

In this blog post, I provide a kickstarter guide to our recently released TrajNet++ framework for human trajectory forecasting. We recently released TrajNet++ Challenge for agent-agent based trajectory forecasting. Details regarding the challenge can be found [here](https://www.aicrowd.com/challenges/trajnet-a-trajectory-forecasting-challenge). This post will focus on utilizing the TrajNet++ framework for easily creating datasets and learning human motion forecasting models.

Overview
========

On a high-level, Trajnet++ constitutes four primary components:

1. [Trajnetplusplustools](https://github.com/vita-epfl/trajnetplusplustools): This repository provides helper functions for trajectory prediction. For instance: trajectory categorization, evaluation metrics, prediction visualization. 

2. [Trajnetplusplusdataset](https://github.com/vita-epfl/trajnetplusplusdataset): This repository provides scripts to generate train, val and test dataset splits from raw data as well as simulators.

3. [Trajnetplusplusbaselines](https://github.com/vita-epfl/trajnetplusplusbaselines): This repository provides baseline models (handcrafted as well as data-driven) for human motion prediction. This repository also provides scripts to extensively evaluate the trained models.

4. [Trajnetplusplusdata](https://github.com/vita-epfl/trajnetplusplusdata): This repository provides the already processed real world data as well as synthetic datasets conforming to human motion. 

Getting Started
===============

I describe how to get started using TrajNet++ with the help of a running example. 
We will create a synthetic dataset using ORCA simulator and train an LSTM-based model to perform trajectory prediction. 

Setting Up Repositories
-----------------------

The first step is to setup the repositories, namely Trajnetplusplusdata for dataset generation and Trajnetplusplusbaselines for model training. Next, we setup the virtual environment and download the requirements. 

```bash
## Create directory to setup Trajnet++
mkdir trajnet++
cd trajnet++ 

## Clone Repositories
git clone https://github.com/vita-epfl/trajnetplusplusdataset.git
git clone https://github.com/vita-epfl/trajnetplusplusbaselines.git

## Make virtual environment
virtualenv -p /usr/bin/python3.6 trajnetv
source trajnetv/bin/activate

## Download Requirements
cd trajnetplusplusbaselines/ 
pip install -e .

cd ../trajnetplusplusdataset/ 
pip install -e .
pip install -e '.[test, plot]'
```

Alright, our repositories are now setup! 

Dataset Preparation
-------------------

[Trajnetplusplusdataset](https://github.com/vita-epfl/trajnetplusplusdataset) helps in creating the dataset splits to train and test our prediction models. In this example, we will be using the ORCA simulator for generating our synthetic data. 
Therefore, we will setup the simulator with the help of this [wonderful repo](https://github.com/sybrenstuvel/Python-RVO2).

```bash
## Download Repository
wget https://github.com/sybrenstuvel/Python-RVO2/archive/master.zip
unzip master.zip
rm master.zip

## Setting up ORCA (steps provided in the Python-RVO2 repo)
cd Python-RVO2-master/
pip install cmake
pip install cython
python setup.py build
python setup.py install
cd ../
```

We also download the Social Force simulator available at [this repository](https://github.com/svenkreiss/socialforce). 

```bash
## Download Repository
wget https://github.com/svenkreiss/socialforce/archive/master.zip
unzip master.zip
rm master.zip

## Setting up Social Force
cd socialforce-master/
pip install -e .
cd ../
```

Now, we will generate controlled data using the ORCA simulator. We will generate 1000 scenarios of 5 pedestrains moving in an interactive setting.

```python
## Destination to store generated trajectories
mkdir -p data/raw/controlled
python -m trajnetdataset.controlled_data --simulator 'orca' --num_ped 5 --num_scenes 1000

## To know more options of generating controlled data
python -m trajnetdataset.controlled_data --help
```

By default, the generated trajectories will be stored in _'orca\_circle\_crossing\_5ped\_1000scenes\_.txt'_. Procedure for extracting publicly available datasets can be found [here](https://github.com/vita-epfl/trajnetplusplusdataset/blob/master/README.rst). Also, the goals of the generated trajectories are stored in the _'goal_files'_ folder under the same name as the .txt file.

We will now convert the generated '.txt' file into the TrajNet++ data structure format. Moreover, we will choose to select only interacting scenes (Type III) from our generated trajectories. More details regarding our data format and trajectory categorization can be found on our [challenge overview page](https://www.aicrowd.com/challenges/trajnet-a-trajectory-forecasting-challenge).

For conversion, open the _trajnetdataset/convert.py_, comment the real dataset conversion part in main() and uncomment the below given snippet.

```python
## Run the conversion
python -m trajnetdataset.convert --linear_threshold 0.3 --acceptance 0 0 1.0 0 --synthetic

## To know more options of converting data
python -m trajnetdataset.convert --help
```

Once the conversion process completes, your converted datasets will be available in the _output_ folder. Trajnetplusplustools provides the following utilities to understand your dataset better. To visualize trajectories in terminal in MacOS, I use [itermplot](https://github.com/daleroberts/itermplot).

```python
## obtain new dataset statistics
python -m trajnetplusplustools.dataset_stats output/train/*.ndjson

## visualize sample scenes
python -m trajnetplusplustools.trajectories output/train/*.ndjson --random

## visualize interactions (Default: Collision Avoidance)
mkdir interactions
python -m trajnetplusplustools.visualize_type output/train/*.ndjson
```

Finally, move the converted data and goal files (if necessary) to the trajnetbaselines folder.

```bash
mv output ../trajnetplusplusbaselines/DATA_BLOCK/synth_data
mv goal_files/ ../trajnetplusplusbaselines/
cd ../trajnetplusplusbaselines/
```
Now that the dataset is ready, its time to train the model! :)

Training Models
---------------

Training models is more easier than generating datasets in Trajnet++ !
All you got to do is ....
```python
python -m trajnetbaselines.lstm.trainer --path synth_data
```
.... and your LSTM model starts training. Your model will be saved in the _synth\_data_ folder within _OUTPUT\_BLOCK_. Currently, models are being saved according to the type of interaction models being used.

In order to train using interaction modules (eg. nearest-neighour encoding) utilizing goal information, run
```python
python -m trajnetbaselines.lstm.trainer --path synth_data --type 'nn' --goals
```

```python
## To know more options about trainer 
python -m trajnetbaselines.lstm.trainer --help
```


Evaluating Models
-----------------

One strength of TrajNet++ is its extensive evaluation system. You can read more about it in the [metrics section here](https://www.aicrowd.com/challenges/trajnet-a-trajectory-forecasting-challenge).

To perform extensive evaluation of your trained model. The results are saved in Results.png 
```python
python -m evaluator.trajnet_evaluator --path synth_data --output OUTPUT_BLOCK/synth_data/lstm_vanilla_None.pkl

## To know more options about evaluator 
python -m evaluator.trajnet_evaluator --help
```
To know more about how the evaluation procedure works, please refer to this [README](https://github.com/vita-epfl/trajnetplusplusbaselines/blob/master/evaluator/README.rst).


Visualize Models
----------------

Visualize learning curves of two different models
```python
python -m trajnetbaselines.lstm.plot_log OUTPUT_BLOCK/synth_data/lstm_vanilla_None.pkl.log OUTPUT_BLOCK/synth_data/lstm_goals_nn_None.pkl.log
```

Visualize predictions of models
```python
# python -m evaluator.visualize_predictions <ground_truth_file> <prediction_file>
python -m evaluator.visualize_predictions DATA_BLOCK/synth_data/test_private/orca_five_synth.ndjson DATA_BLOCK/synth_data/test_pred/lstm_vanilla_None_modes1/orca_five_synth.ndjson --n 10 --random
```

Done Done
=========

I hope this blog provides you with the necessary kickstart for using TrajNet++. If you have any questions, feel free to post issues on [Github](https://github.com/vita-epfl/trajnetplusplusbaselines). If you liked using TrajNet++, a token of appreciation to parth.kothari@epfl.ch would really go a long way for me ! :)

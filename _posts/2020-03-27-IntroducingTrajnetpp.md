---
title: 'Introdcuing TrajNet++ : A Framework for Human Trajectory Prediction'
date: 2020-03-27
permalink: /posts/2020/03/intro_trajnetpp/
tags:
  - trajectory prediction
  - autonomous systems
  - social intelligence
---

In this blog post, I provide a kickstart guide to our recently released TrajNet++ framework for human trajectory prediction. We recently released TrajNet++ Challenge for doing agent-agent based trajectory prediction as part of [ICRA workshop on Long Term Human Motion Perediction](https://motionpredictionicra2020.github.io). Details regarding the challenge can be found [here](https://www.aicrowd.com/challenges/trajnet-a-trajectory-forecasting-challenge). This post will focus on utilizing the TrajNet++ framework for easily creating datasets and learning models human motion prediction.

Overview
========

On a high-level, Trajnet++ constitutes of four primary components:

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

```python
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
Therefore, we will setup the simulator with the help of this [wonderful repo](https://github.com/sybrenstuvel/Python-RVO2)

```python
## Download Repository
wget https://github.com/sybrenstuvel/Python-RVO2/archive/master.zip
unzip master.zip
rm master.zip

## Setting up ORCA (steps provided in the Python-RVO2 repo)
cd Python-RVO2-master/
pip install cython
python setup.py build
python setup.py install
cd ../
```

We also download the Social Force simulator available at [this repository](https://github.com/svenkreiss/socialforce). 

```python
## Download Repository
https://github.com/svenkreiss/socialforce/archive/master.zip
unzip master.zip
rm master.zip

## Setting up Social Force
cd socialforce-master/
pip install -e .
cd ../
```

Now, we will generate controlled data using the ORCA simulator. We will generate 100 scenarios of 6 pedestrains in a circle_crossing (default) setting.

```python
## Destination to store generated trajectories
mkdir -p data/raw/controlled
python -m trajnetdataset.controlled_data --simulator 'orca' --num_ped 10 --num_scenes 100

## To know more options of generating controlled data
python -m trajnetdataset.controlled_data --help
```

By default, the generated trajectories will be stored in _'orca\_circle\_crossing\_6ped\_.txt'_. Procedure for extracting publicly available datasets can be found [here](https://github.com/vita-epfl/trajnetplusplusdataset/blob/master/README.rst)

We will now convert the generated '.txt' file into the TrajNet++ data structure format. Moreover, we will choose to select only interacting scenes (Type III) from our generated trajectories. More details regarding our data format and trajectory categorization can be found on our [challenge overview page](https://www.aicrowd.com/challenges/trajnet-a-trajectory-forecasting-challenge)

For conversion, open the _trajnetdataset/convert.py_, comment the real dataset conversion part in main() and uncomment the below given snippet
```python
## Comment the real dataset conversion part in main()

## Uncomment the following snippet
write(controlled(sc, 'data/raw/controlled/orca_circle_crossing_6ped_.txt'),
      'output_pre/{split}/orca_circle_crossing_6ped.ndjson', args)
categorize(sc, 'output_pre/{split}/orca_circle_crossing_6ped.ndjson', args)

## Run the conversion
python -m trajnetdataset.convert --linear_threshold 0.3 --acceptance 0 0 1.0 0

## To know more options of converting data
python -m trajnetdataset.convert --help
```

Once the conversion process completes, your converted datasets will be available in the _output_ folder. Trajnetplusplustools provides the following utilities to understand your dataset better. To visualize trajectories in terminal in MacOS, I use [itermplot](https://github.com/daleroberts/itermplot)

```python
## obtain new dataset statistics
python -m trajnettools.dataset_stats output/train/*.ndjson

## visualize sample scenes
python -m trajnettools.trajectories output/train/*.ndjson --random
```

Finally, move the converted data to the trajnetbaselines folder.
```python
mv output ../trajnetplusplusbaselines/DATA_BLOCK/synth_data
cd ../trajnetplusplusbaselines/
```
Now that the dataset is prepared, its time to train the model! :)

Training Models
---------------

Training models is more easier than generating datasets in Trajnet++ !
All you got to do is ....
```python
python -m trajnetbaselines.lstm.trainer --path synth_data
```
.... and your LSTM model starts training. Your model will be saved in the _synth\_data_ folder within _OUTPUT\_BLOCK_. Currently, models are being saved according to the type of interaction models being used.

```python
## To know more options about trainer 
python -m trajnetbaselines.lstm.trainer --help
```

Evaluating Models
-----------------

One strength of TrajNet++ is its extensive evaluation system. You can read more about it in the [metrics section here](https://www.aicrowd.com/challenges/trajnet-a-trajectory-forecasting-challenge)

To perform extensive evaluation of your trained model. The results are saved in Results.png 
```python
python -m evaluator.trajnet_evaluator --data synth_data --output OUTPUT_BLOCK/synth_data/vanilla.pkl

## To know more options about evaluator 
python -m evaluator.trajnet_evaluator --help
```
To know more about how the evaluation procedure works, please refer to this [README](https://github.com/vita-epfl/trajnetplusplusbaselines/blob/master/evaluator/README.rst)

Done Done
=========

I hope this blog provides you with the necessary kick-start of using TrajNet++. If you have any questions, feel free to post issues on [Github](https://github.com/vita-epfl/trajnetplusplusbaselines). If you liked using TrajNet++, a token of appreciation to parth.kothari@epfl.ch would really go a long way for me ! :)
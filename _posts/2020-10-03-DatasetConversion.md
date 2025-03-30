---
title: 'TrajNet++ : Dataset Conversion'
date: 2020-10-03
permalink: /posts/2020/10/data_conversion/
tags:
  - trajectory forecasting
  - data conversion
  - social intelligence
---

*Note: TrajNet++ is no longer actively maintained.*

In this blog post, I provide a quick tutorial to converting external datasets into the desired [.ndjson](http://ndjson.org/) format using the TrajNet++ framework. This post will focus on utilizing the TrajNet++ dataset code for easily converting new datasets.

<!-- Overview
========

On a high-level, Trajnet++ dataset code does three primary functions:

1. [readers.py](https://github.com/vita-epfl/trajnetplusplusdataset/blob/master/trajnetdataset/readers.py): This code converts the raw dataset files into trackrows in .ndjson format

2. [scene.py](https://github.com/vita-epfl/trajnetplusplusdataset/blob/master/trajnetdataset/scene.py): This code constructs the different scenes (i.e. the dataset samples), as per the specification provided, from the trackrows.

3. [get_type.py](https://github.com/vita-epfl/trajnetplusplusdataset/blob/master/trajnetdataset/get_type.py): This code performs trajectory categorization for every scene. Read more about our trajectory categorization [here](https://arxiv.org/pdf/2007.03639v2.pdf).  -->


Details
=======

In this tutorial, I will convert the ETH dataset utilized by the [Social GAN](https://github.com/agrimgupta92/sgan) paper.


Step 1: Downloading Data
------------------------

Before proceeding, please setup the base repositories. See 'Setting Up Repositories' [here](https://thedebugger811.github.io/posts/2020/03/intro_trajnetpp/)

```bash
## Checkout 'eth' branch of Trajnetplusplusdataset
git checkout -b eth origin/eth

## Download external data
sh download_data.sh
cp -r datasets/eth/ data/

```

Step 2: Converting Raw Data
---------------------------

1. In our external dataset, each trajectory point is delimited by '\t'
2. TrackRow takes the arguments 'frame', 'ped_id', 'x', 'y' in order.

```python
def standard(line):
    line = [e for e in line.split('\t') if e != '']
    return TrackRow(int(float(line[0])),
                    int(float(line[1])),
                    float(line[2]),
                    float(line[3]))
```

Code snippet already provided in [readers.py](https://github.com/vita-epfl/trajnetplusplusdataset/blob/eth/trajnetdataset/readers.py)

Dataset Generation and Categorization
-------------------------------------

For dataset conversion, we call the 'raw dataset conversion' code shown above in convert.py

```python
def standard(sc, input_file):
    print('processing ' + input_file)
    return (sc
            .textFile(input_file)
            .map(readers.standard)
            .cache())
```

Code snippet already provided in convert.py

Finally, we call the appropriate data files for conversion and categorization (See [convert.py](https://github.com/vita-epfl/trajnetplusplusdataset/blob/eth/trajnetdataset/convert.py)).

```bash
python -m trajnetdataset.convert --obs_len 8 --pred_len 12
```

Now that the dataset is ready [in _output_ folder], you can train the model! :)

Difference in generated data
============================

1. Partial tracks are now included (for correct occupancy maps)
2. Pedestrians that appear in multiple chunks had the same id before (might be a problem for some input readers)
3. Explicit index of scenes with annotation of the primary pedestrian

Summarizing
===========

So, for converting any external dataset, all you got to do is 4 simple steps:
1. Download data and place it in the /data folder. 
2. Edit [readers.py](https://github.com/vita-epfl/trajnetplusplusdataset/blob/master/trajnetdataset/readers.py) to convert raw format into TrackRows in [.ndjson](http://ndjson.org/) format
3. Call the above snippet in [convert.py](https://github.com/vita-epfl/trajnetplusplusdataset/blob/master/trajnetdataset/convert.py)
4. Call the dataset generation code with the appropriate arguments.



We recently released TrajNet++ Challenge for agent-agent based trajectory forecasting. Details regarding the challenge can be found [here](https://www.aicrowd.com/challenges/trajnet-a-trajectory-forecasting-challenge).


---
layout: page
title: Transfer Learning in Deep Q-Networks
description: project blog
---
### Cartesius

For this project I run all programs on the [Dutch national supercomputer](https://userinfo.surfsara.nl/systems/cartesius) called Cartesius. You need a [SURFsara](https://www.surf.nl/en/about-surf/subsidiaries/surfsara) account to be able to do this. 

I logged into my account on Cartesius and cloned a [Theano-based implementation of Deep Q-Learning](https://github.com/spragunr/deep_q_rl):

```
ssh username@doornode.surfsara.nl
git clone https://github.com/spragunr/deep_q_rl
cd deep_q_rl
```

Now we are inside the `deep_q_rl/` directory which has all the files that we need. Next, we want to create a `job.sh` file that contains the instructions of what we want to run:

```
#!/bin/bash
#SBATCH -p gpu
#SBATCH -N 1
#SBATCH -t 120:00:00
module load cuda
module load cudnn
module load torch7
module load opencv/gnu/2.4.10
module load python/2.7.11
THEANO_FLAGS='device=gpu,floatX=float32' srun -u python deep_q_rl/run_nips.py --rom pong.bin
```

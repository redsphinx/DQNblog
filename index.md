---
layout: page
title: Transfer Learning in Deep Q-Networks
description: project blog
---

### Training on Cartesius

For this project I run all programs on the [Dutch national supercomputer](https://userinfo.surfsara.nl/systems/cartesius) called Cartesius. You need a [SURFsara](https://www.surf.nl/en/about-surf/subsidiaries/surfsara) account to be able to do this. 

I logged into my account on Cartesius and cloned a [Theano-based implementation of Deep Q-Learning](https://github.com/spragunr/deep_q_rl):

```
ssh username@doornode.surfsara.nl
git clone https://github.com/spragunr/deep_q_rl
cd deep_q_rl
```

Now we are inside the `deep_q_rl/` directory which has all the files that we need. Next, we want to create a `job.sh` file that contains the instructions of what we want to run:

```
vim job.sh
```

Paste this in the file:

```
#!/bin/bash
#SBATCH -p gpu
#SBATCH -N 1
#SBATCH -t 40:00:00
module load cuda
module load cudnn
module load torch7
module load opencv/gnu/2.4.10
module load python/2.7.11
THEANO_FLAGS='device=gpu,floatX=float32' srun -u python deep_q_rl/run_nips.py --rom pong.bin
```

Save and quit the file. Then make this file executable by running:

```
chmod +x job.sh
```

Next we need to modify the original `dep_script.sh` to something we can run. We copy the code involving the install of `pylearn2` and `ALE` and paste this a new file called `install_dep.sh`. Now `install_dep.sh` looks like this:

```
#!/bin/bash
git clone git://github.com/lisa-lab/pylearn2.git
cd ./pylearn2
python setup.py develop --user
cd ..
git clone https://github.com/mgbellemare/Arcade-Learning-Environment.git ALE
cd ./ALE
mkdir build && cd build
cmake -DUSE_SDL=ON -DUSE_RLGLUE=OFF -DBUILD_EXAMPLES=ON ..
make -j 4
pip install --user
cd ..

```

Then we make this file executable by running the same `chmod` command:

```
chmod +x install_dep.sh
```

Now we're ready to install `pylearn2` and `ALE`. Let's do it:

```
module load cuda cudnn torch7 opencv/gnu/2.4.10 python/2.7.11
./install_dep.sh
```

Next, navigate to the `ALE/` directory that was just created and copy the file `ale.cfg` and paste it in the main `deep_q_rl/` directory:

```
cd ALE/
cp ale.cfg ../
```

We're almost there. Now navigate to the `roms/` directory. There is a link for ROMs that you need to download. So for example, in our case we want to train the DQN to play Pong, so we need the Pong ROM file. However, the code we originally cloned from Github doesn't know that the game was originally called **Video Olympics** and neither did I. It took some time to figure this out, so be sure of what the name was called originally and also check how the code thinks it is called by checking the supported games in:

```
cd ~/deep_q_rl/ALE/src/games/supported/
```

In here you can list all the cpp and hpp files that the code needs to be able to run an Atari 2600 ROM. If the cpp/hpp file is not there, you either need to accept that the game isn't supported or try to build your own cpp/hpp files. I don't have any experience with this. So we can see that for example `Pong.hpp` exists. If we open this file, with `vim` for example, we can see that it contains the line: 

```
// the rom-name
const char* rom() const { return "pong"; }
```

This is the name you need to feed in the following 2 files to be able to run the code. Do:

```
vim ~/deep_q_rl/job.sh
```

and edit the line so it says:

```
THEANO_FLAGS='device=gpu,floatX=float32' srun -u python deep_q_rl/run_nips.py --rom pong.bin
```

Then we need to set the `BASE_ROM_PATH` so the program knows where your ROM files are located and we need to change the name of the ROM to "pong":

```
vim ~/deep_q_rl/deep_q_rl/run_nips.py
```

and edit the line so it says:

```
BASE_ROM_PATH = "/home/username/deep_q_rl/roms/"
ROM = 'pong.bin'
```

Now we are ready:

```
cd ~/deep_q_rl/
sbatch job.sh
squeue -u <username>
```

If everything is running smooth, the output should look something like this:

```
  JOBID PARTITION     NAME       USER ST        TIME  NODES NODELIST(REASON)
2099600       gpu   job.sh   username  R    00:00:05      1 gcn18

```
After the job has finished running, you have a trained model!! Hooray!!
If we check out the contents of ```deep_q_rl/``` we can see that there is a new folder called ```pong_xx-xx-xx-xx_xxxxxx_xxxx```, the x's being numbers that get automatically generated. This folder contains everything about our trained model.

I will rename this folder ```pong-nips```:

```
mv ~/deep_q_rl/pong__xx-xx-xx-xx_xxxxxx_xxxx ~/deep_q_rl/pong-nips
```

### Playing on Cartesius

After about a day you should have a trained network for your game. (For the nature paper it takes about 4 to 5 days on Cartesius) Let's watch it play!
Log in to your SURFsara account through the terminal like this:

```
ssh -CY username@cartesius.surfsara.nl
```

Note that you have to be connecting from a whitelisted IP-address. You can send an e-mail to their [helpdesk](https://userinfo.surfsara.nl/contact) to get your IP whitelisted. 

Make a ```.theanorc``` in your Cartesius home directory containing the following:

```
[global]
floatX = float32
device = gpu
allow_gc = False
```

We also need to make a ```sleepjob.sh``` file in order to reserve a GPU node to later execute python code on:

```
#!/bin/bash
#SBATCH -p gpu
#SBATCH -N 1
#SBATCH -t 1:00:00
sleep 1200
```

All this does is sleep and reserving a node for us to work on. Next, run it:

```
sbatch sleepjob.sh
```

Now go to:

```
cd ~/deep_q_rl/deep_q_rl/
vim ale_run_watch.py
```

Because we trained using the ```run_nips.py``` file we also need to run it with that file so make the change:

```
:s/run_nature/run_nips/
:wq
```

Now run ```squeue``` to get the node where ```sleepjob.sh``` is running on:

```
squeue -u username
```

This returns: 

```
  JOBID PARTITION       NAME       USER ST        TIME  NODES NODELIST(REASON)
2099600       gpu   sleepjob   username  R    00:00:05      1 gcn18

```

And we need ```gcn18```. Now ssh with -CY parameters into this node:

```
ssh -CY gcn18
```

Now load the modules necessary:

```
module load cuda cudnn torch7 opencv/gnu/2.4.10 python/2.7.11
```

And run the file:

```
cd ~/deep_q_rl/deep_q_rl/
python ale_run_watch.py ../pong-nips/network_file_99.pkl
```

And voila, you can see your trained network playing Pong!!!
![](http://i.imgur.com/OADmd83.gif)

To see a graph with training stuff run:

```
python plot_results.py ../pong-nips/results.csv
```

Which gives:

![](http://i.imgur.com/YuhzDaz.png)

To visualize the filters in the first layers of the trained network:

```
python plot_filters.py python ../pong-nips/network_file_99.pkl 
```

which gives:

![](http://i.imgur.com/69U4MVl.png)

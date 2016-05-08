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
module load cuda cudnn torch7 module load opencv/gnu/2.4.10 module load python/2.7.11
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

### Playing with a trained model

All of our files are now on Cartesius. So we need to move them to a local computer so we can watch it play. To do this I will let it run on a virtual machine on my laptop. The reason I don't install it directly on my laptop is because in the past I've messed up quite a lot by experimenting with my system. I can only recommend using virtual machines. 

#### VirtualBox
Let's start by opening up a terminal on our local computer and installing [VirtualBox](https://www.virtualbox.org/manual/UserManual.html):

```
sudo apt-get install virtualbox
```

Next, download a [Ubuntu 14.04 64-bit desktop ISO image](http://releases.ubuntu.com/14.04/ubuntu-14.04.4-desktop-amd64.iso). Next, launch VirtualBox and make a new Linux machine. I made mine with the following specifications:

```
Drive size: 25 GB
RAM: 4096 MB
Video Memory: 128 MB
CPUs: 2
```

I will refer to the virtual machine as VM from now on. Start the VM and install Ubuntu. I installed mine with 4096 MB of swap space. When that is installed, click on ```Devices``` and choose ```Install Guest Additions``` and restart the VM. 

From this point on we will work on the VM.

#### Installing deep_q_rl
First install the very basics that we will need:

```
sudo apt-get install vim
sudo apt-get install git
```

Next, clone [spragunr's](https://github.com/spragunr/deep_q_rl) repository to the VM -- this is the repo we downloaded way at the beginning on Cartesius.

```
cd Documents/
git clone https://github.com/spragunr/deep_q_rl
```

Then open ```dep_script.sh``` file:

```
cd deep_q_rl/
vim deep_q_rl/dep_script.sh
```

and then we make a small change with this regular expression:

```
:%s/make -j2/make -j 4/
:wq
```

Now we run the ```dep_script.sh```:

```
./deep_q_rl/dep_script.sh
```

#### Installing CUDA


#### Getting the files on Cartesius


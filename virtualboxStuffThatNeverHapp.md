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
vim dep_script.sh
```

and then we make a small change with this regular expression:

```
:%s/make -j2/make -j 4/
:wq
```

Now we run the ```dep_script.sh```:

```
./dep_script.sh
```

#### Don't forget the ROM
Go to [this website](http://www.atariage.com/system_items.html?SystemID=2600&ItemTypeID=ROM) and get the pong ROM:

```
cd roms/
wget http://atariage.com/2600/roms/VideoOlympics.zip
unzip VideoOlympics.zip
mv VideoOlympics.bin pong.bin
```

#### Installing CUDA
Download and install CUDA:

```
cd ~/Downloads
wget http://developer.download.nvidia.com/compute/cuda/7.5/Prod/local_installers/cuda-repo-ubuntu1404-7-5-local_7.5-18_amd64.deb
md5sum cuda-repo-ubuntu1404-7-5-local_7.5-18_amd64.deb
```

The last statement returns a long string that should match this: ```5cf65b8139d70270d9234d5ff4d697c7```

Now install CUDA:

```
sudo dpkg -i cuda-repo-ubuntu1404-7-5-local_7.5-18_amd64.deb
sudo apt-get update
sudo apt-get install cuda
```

#### Getting the files on Cartesius

At the end of the **Cartesius** part of this tutorial, we had a folder named ```pong_xx-xx-xx-xx_xxxxxx_xxxx``` with all the files we need. Log in on Cartesius. This can be done in the terminal of the VM or the terminal of your real local machine. It doesn't matter. I set up a new repository on my GitHub account and pushed the ```pong``` folder. For instructions on how to do this see [this](https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/). Note that you have to make the following changes through commandline before being able to push:

```
unset SSH_ASKPASS
git remote set-url origin https://Your-Github-UserName@github.com/Your-Github-UserName/REPO-NAME
```

Now we go to the terminal of our VM and we clone the folder:

```
cd Documents/
git clone https://github.com/Your-Github-UserName/REPO-NAME
```

We will keep calling this folder **REPO-NAME**, but note that you will probably call it something else. 

#### Pressing Play

The moment of truth, either it works or it doesn't. 

First try out something easy on the VM:

```
cd Documents/deep_q_rl/deep_q_rl/
python plot_results.py ../../REPO-NAME/pong_xx-xx-xx-xx-...-xxx/results.csv
```

If everything went well we can see a graph with learning stuff. 

Now the hard part:

```

```
So this doesn't work clearly but I put some time and effort into logging this stuff so I will somehow make them into some other part of a blog ...ever maybe

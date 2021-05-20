# PyProteus
My personal notes about installing and running python scripts on PROTEUS supercluster.


## SSH to PROTEUS
```bash
ssh -X USER@proteus.ugr.es
```
## Install openssl for pip on user folder
from: https://github.com/actions/setup-python/issues/93

First install openssl, in [this page](https://help.dreamhost.com/hc/en-us/articles/360001435926-Installing-OpenSSL-locally-under-your-username) I found the following instructions:

```bash
wget https://www.openssl.org/source/openssl-1.1.1k.tar.gz
tar -zxvf openssl-1.1.1k.tar.gz
cd openssl-1.1.1k
./config --prefix=/home/$USER/openssl --openssldir=/home/$USER/openssl no-ssl2
make
make test
make install
cd ~
```

Edit the file `.bash_profile` with vim or nano:

```bash
nano ~/.bash_profile
```

And append the following lines:

```bash
export PATH=$HOME/openssl/bin:$PATH
export LD_LIBRARY_PATH=$HOME/openssl/lib
export LC_ALL="en_US.UTF-8"
export LDFLAGS="-L/home/$USER/openssl/lib -Wl,-rpath,/home/$USER/openssl/lib"
```

Finally:

```bash
source ~/.bash_profile
```

Test to confirm it's installed in the correct location and that the version is 1.1.1:

```bash
which openssl
openssl version
```

Only if everything worked:
```bash
rm -r openssl-1.1.1k
```

## Install Python on user folder
from: https://www.benhepworth.com/blog/2016/05/18/install-python-to-separate-directory-on-linux-in-5-easy-steps/

Get the last python3 source files:
```bash
wget https://www.python.org/ftp/python/3.9.5/Python-3.9.5.tar.xz
tar -zxvf Python-3.9.5.tar.xz
cd Python-3.9.5
```

Configure our installation with user paths:
```bash
./configure --prefix=/home/$USER/python3 --enable-optimizations --with-openssl=/home/$USER/openssl
```
### Compile and install
```bash
make
make test (Optional. More than 2 hours, skip if needed.)
make install
```

### Add to PATH:
Edit the file `.bashrc` with vim or nano:
```bash
nano ~/.bashrc
```

And append the following lines:

```bash
export PATH="${PATH}:/home/${USER}/python3/bin"
```

### Verification
Now, test it:

```bash
source ~/.bashrc
python3 --version
```

If everything worked, you should be able to see the downloaded version of python3.
Only if everything worked:
```bash
rm -r ~/Python-3.9.5
```

### Updating pip
```bash
pip3 install --upgrade pip
```


### Create virtual environment
```bash
cd ~
python3 -m venv env_astro
source env_astro/bin/activate
```

### Install needed libraries
```bash
pip3 install example_package
```

## Run your code
Create a file named `spython3.sh`with the following content:
```bash
echo -e "#!/bin/bash\npython3 \$1\n" > pythonlauncher.sh
echo -e "Launching $1 ..."

# For 10 cores and 10000 units of memory, without email notification:
slanzarv -c 10 -m 10000 --nomail sh pythonlauncher.sh $1

# For standard configuration:
#slanzarv sh pythonlauncher.sh $1
```

And give tthe proper permissions:
```bash
chmod +x spython3.sh
```


Now, you can call your python3 scripts with:
```bash
./spython3.sh yourScript.py
```

Once the process is completed, you car remove the file `pythonlauncher.sh`. For view the status of your task, you can type `sacct`, or if you want to see the process in real time, use `watch sacct`.

For view in real time the output of your code, type `tail -f sh.YourProcessIDhere.out`.

Note: for my comfort, I made the following alias to my `.bashrc`:
```bash
nano ~/.bashrc
```

And appending the following line:

```bash
alias spython3="~/spython3.sh"
```

I'm able to run python3 code in my own folder, as usual, and running the same code in the supercluster with this simple notation:
```bash
python3 yourScript.py  # For normal python3 use
spython3 yourScript.py # For using the supercluster
```

I hope it can help you!

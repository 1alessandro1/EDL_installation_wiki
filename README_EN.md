# EDL_installation_wiki

This guide exists because I always forget how to install Bjoern Kerler’s [awesome project](https://github.com/bkerler/edl).

We will use EDL with **pyenv**, with **virtualenv**, and **Python 3.8**, which does not cause issues like newer versions for this project.

## 1. System prerequisites

Install the packages needed to build Python from source:

```bash
sudo apt update
sudo apt install -y \
    build-essential \
    libssl-dev \
    zlib1g-dev \
    libbz2-dev \
    libreadline-dev \
    libsqlite3-dev \
    libncurses5-dev \
    libncursesw5-dev \
    xz-utils \
    tk-dev \
    libffi-dev \
    liblzma-dev \
    uuid-dev \
    libdb-dev \
    curl \
    git \
    zsh
```

---

## 2. Installing `pyenv` and `pyenv-virtualenv`

Official procedure:

```bash
# pyenv installation
curl https://pyenv.run | bash
```

Add this to your `.zshrc`:

```zsh
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init - zsh)"
eval "$(pyenv virtualenv-init - zsh)"
```

Then reopen the terminal or run:

```zsh
source ~/.zshrc
```

Only if it still gives problems, try:

```zsh
exec "$SHELL"
```

---

## 3. Install a specific Python version

For example, Python 3.8.18:

```bash
pyenv install 3.8.18
```

---

## 4. Create a dedicated virtualenv for EDL

```bash
pyenv virtualenv 3.8.18 qualcomm-EDL
```

This will create an isolated environment for the EDL project.

---

## 5. Use the virtualenv locally only in `/home/$(whoami)/qualcomm-EDL`

Go into the project folder:

```bash
mkdir /home/$(whoami)/qualcomm-EDL && cd /home/$(whoami)/qualcomm-EDL
pyenv local qualcomm-EDL
```

This creates a `.python-version` file and ensures that **only in this folder** Python uses the virtualenv.
Outside this folder, Python will remain the system one (`/usr/bin/python3`).

Check:

```bash
cd ~
which python3
# /usr/bin/python3

cd /home/$(whoami)/Qualcomm_EDL
which python3
# /home/ale/.pyenv/shims/python3
```

---

## 6. Installing EDL

Debian/Ubuntu/Mint/etc.

```bash
sudo apt install adb fastboot python3-dev python3-pip liblzma-dev git && sudo apt purge modemmanager
git clone https://github.com/bkerler/edl.git # do NOT use --recurse-submodules
cd edl
git submodule update --init --recursive
pip3 install -r requirements.txt
```

Proceed by running the script as root:

```bash
chmod +x ./install-linux-edl-drivers.sh
bash ./install-linux-edl-drivers.sh
sudo update-initramfs -u
```

## 7. Reboot; after reboot build the project

```bash
python3 setup.py build
sudo $(which python3) setup.py install
```

# Common issues

`pylzma` may fail to build if GCC is too new. On Ubuntu 24 or later where `gcc-14` is present, you need to install an older version:

```bash
sudo apt install gcc-12 g++-12
export CC=gcc-12
export CXX=g++-12
pip install pylzma==0.5.0
```

# End

Now, when you enter the path where EDL was installed, it will use its Python 3.8 and the virtualenv will be automatically activated.

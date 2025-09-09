# EDL_installation_wiki
Questa guida esiste perché mi dimentico sempre come fare a installare il [progetto fantastico](https://github.com/bkerler/edl) di Bjoern Kerler 

Useremo EDL con pyenv, con virtualenv e Python 3.8/3.9.

## 1. Prerequisiti di sistema

Installa i pacchetti necessari per buildare Python da sorgente:

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
    libdb-dev
```

---

## 2. Installazione di `pyenv` e `pyenv-virtualenv`

Procedura ufficiale:

```bash
# installazione pyenv
curl https://pyenv.run | bash
```

Aggiungi nel tuo `.zshrc`:

```zsh
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init - zsh)"
eval "$(pyenv virtualenv-init - zsh)"
```

Poi riapri il terminale o fai:

```bash
source ~/.zshrc
```

Solo se ancora da problemi, prova con `exec "$SHELL"`

---

## 3. Installare una versione specifica di Python

Ad esempio, Python 3.8.18:

```bash
pyenv install 3.8.18
```

---

## 4. Creare un virtualenv dedicato a EDL

```bash
pyenv virtualenv 3.8.18 qualcomm-EDL
```

Questo creerà un ambiente isolato per il progetto EDL.

---

## 5. Uso locale del virtualenv solo in `/home/ale/Qualcomm_EDL`

Vai nella cartella del progetto:

```bash
cd /home/ale/Qualcomm_EDL
pyenv local qualcomm-EDL
```

Questo crea un file `.python-version` e fa sì che **solo in questa cartella** il Python usi il virtualenv.
Fuori da questa cartella, Python continuerà a essere quello di sistema (`/usr/bin/python3`).

Controllo:

```bash
cd ~
which python3
# /usr/bin/python3

cd /home/ale/Qualcomm_EDL
which python3
# /home/ale/.pyenv/shims/python3
```

---


## 6. Installazione di EDL

Debian/Ubuntu/Mint/etc

```
sudo apt install adb fastboot python3-dev python3-pip liblzma-dev git && sudo apt purge modemmanager
git clone https://github.com/bkerler/edl.git # do NOT use --recurse-submodules
cd edl
git submodule update --init --recursive
pip3 install -r requirements.txt
```

Proseguire con l'eseguire lo script come root:

```
chmod +x ./install-linux-edl-drivers.sh
bash ./install-linux-edl-drivers.sh
sudo update-initramfs -u
```

## 7. Riavviare, al riavvio build del progetto

```
python3 setup.py build
sudo $(which python3) setup.py install
```

# Fine. Ora entrando nel percorso dove è stato installato EDL, verrà utilizzato il suo python3.8 e il virtualenv verrà attivato automaticamente


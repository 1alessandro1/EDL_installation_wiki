# EDL installation wiki
Questa guida esiste perché mi dimentico sempre come fare a installare il [progetto fantastico](https://github.com/bkerler/edl) di Bjoern Kerler.

Useremo EDL con `pyenv`, creando un virtualenv con una versione più vecchia di Python (3.11/3.12) per non avere i problemi di compatibilità causati dalle versioni più recenti (come la 3.13) su questo progetto.

## 1. Prerequisiti di sistema

Installa i pacchetti necessari per buildare Python da sorgente e i tool base:

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
    zsh \
    adb \
    fastboot

# Disinstalla modemmanager perché va in conflitto con le porte seriali/EDL
sudo apt purge modemmanager
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
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init - zsh)"
eval "$(pyenv virtualenv-init - zsh)"
```

O se usi `.bashrc`:

```bash
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init - bash)"
eval "$(pyenv virtualenv-init -)"
```

Chiudi e riapri il terminale, oppure ricarica la shell:

```bash
exec "$SHELL"
```

---

## 3. Installare una versione specifica di Python

Installiamo Python 3.12.13 (o una versione stabile come la 3.12.2) non useremo l'ultima perché `setuptools` nella 3.13.* è stato deprecato.

```bash
pyenv install 3.12.13
```

---

## 4. Creare un virtualenv dedicato a EDL

```bash
pyenv virtualenv 3.12.13 qualcomm-EDL
```

Questo creerà un ambiente isolato basato su quella versione.

---

## 5. Clonare il progetto e attivare il virtualenv

Creiamo la cartella di lavoro, cloniamo il progetto ed entriamo al suo interno.
*(Nota: NON usare `--recurse-submodules` nel clone iniziale)*

```bash
mkdir -p /home/$(whoami)/Qualcomm_EDL && cd /home/$(whoami)/Qualcomm_EDL
git clone https://github.com/bkerler/edl.git
cd edl
git submodule update --init --recursive
```

Ora che siamo dentro la cartella `edl`, leghiamo il virtualenv a questo percorso:

```bash
pyenv local qualcomm-EDL
```

Questo crea un file `.python-version`. D'ora in poi, **solo in questa cartella** (e nelle sue sottocartelle), Python userà il virtualenv isolato.

**Controllo:**
```bash
which python3
# Deve rispondere: /home/tuoutente/.pyenv/shims/python3
```

---

## 6. Installazione delle dipendenze e di EDL

Ora che siamo nel virtualenv, **NON usare `sudo`** per installare i pacchetti Python, altrimenti finiranno nel sistema operativo.

Usiamo `CFLAGS` per evitare che l'installazione di moduli C (come `pylzma`) fallisca sui sistemi Linux più moderni dotati di GCC 14+.

```bash
# Aggiorna pip nel virtualenv
pip install --upgrade pip setuptools wheel

# Installa i requisiti
pip install -r requirements.txt

# Installa il programma superando i blocchi del compilatore C
CFLAGS="-Wno-int-conversion" pip install -U .
```

---

## 7. Installazione dei Driver Linux (Richiede Root)

I driver `udev` sono a livello di sistema operativo e devono essere installati come root.

```bash
chmod +x ./install-linux-edl-drivers.sh
sudo ./install-linux-edl-drivers.sh
sudo update-initramfs -u
```

## 8. Riavvio

Riavvia il computer per rendere effettive le regole udev e l'initramfs:
```bash
sudo reboot
```

***

# Problemi comuni

**Errore di compilazione con `pylzma` (exit code 1):**
Normalmente il prefisso `CFLAGS="-Wno-int-conversion"` utilizzato al Punto 6 risolve il problema sui compilatori moderni (es. Ubuntu 24+ con GCC 14).
Tuttavia, se la compilazione dovesse fallire lo stesso, puoi aggirare il problema forzando l'uso di un compilatore più vecchio:

```bash
# Installa GCC 12
sudo apt install gcc-12 g++-12

# Dì al sistema di usare temporaneamente queste versioni
export CC=gcc-12
export CXX=g++-12

# Reinstalla pylzma e poi EDL
pip install pylzma==0.5.0
pip install -U .
```

# Fine
Ora, ogni volta che entrerai nel percorso `Qualcomm_EDL/edl`, il terminale utilizzerà automaticamente il suo Python 3.8 pulito. Per lanciare il programma ti basterà digitare `edl`.

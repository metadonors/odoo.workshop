---
title: "Precorso"
date: 2018-06-27T16:37:24+02:00
weight: 1
---

Prima di arrivare al corso sarebbe opportuno avere già installati e funzionanti un edito di testo, Git, Docker e Docker Compose

## Editor di testo

Non ci sono preferenze per l'editor di testo, ovviamente è opportuno usarne uno che aiuti a lavorare con codice python e javascript

Un elenco di alcuni possibili editor:

* VSCode [https://code.visualstudio.com/](https://code.visualstudio.com/)
* Atom [https://atom.io/](https://atom.io/)
* Sublime Text [https://www.sublimetext.com/](https://www.sublimetext.com/)

## Git

L'installazione di Git varia in base al sistema operativo utilizzato, ecco come procedere:

### Linux

Su Ubuntu/Debian è sufficiente dare da terminale:

```bash
    $ sudo apt install git
```

Su Redhat/Centos

```bash
    $ sudo yum install git
```

### Mac

Su mac, se non l'avete gia' fatto, prima dell'installazione di Git bisogna procedere a installare [Homebrew](http://brew.sh/) (cosa piuttosto importante se si intende sviluppare software)

Aprite un terminale e copia-incollate il seguente comando:

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew doctor
```

Vi chiedera di installare la _Command Line Developer Tool_ di Apple. Cliccate _Install_ per confermare. Una volta terminato premete _Invio_ per terminare l'installazione di Homebrew.

Effettuato queso passaggio potete installare Git da terminale con il comando:

```
brew install git
```

### Windows

Su Windows scaricate l'applicazione [Git for Windows](https://gitforwindows.org/) e installatela

## Docker e Docker Compose

**Docker** è uno strumento di pacchetizzazione generico per le nostre applicazioni. Semplifica l'installazione di ambienti complessi sia in fase di sviluppo che di produzione. Per noi è utile per riuscire ad avere una piattaforma uguale per tutti su cui lavorare.
Il risultato di una pacchettizzazione con Docker è chiamato _Container_. Un container è a tutti gli effetti un eseguibile che potete lanciare da linea di comando passandogli parametri secondo necessità.

Non è necessario utilizzare la versione Enterprise (EE), utilizzeremo la Community Edition (CE) che ha tutte le funzionalità necessarie (ma senza la stessa assistenza)

**Docker Compose** invece serve a definire e lanciare diversi container in maniera orchestrata. Con compose si utilizza un file di configurazione YAML per definire tutti i servizi di cui è composta la nostra applicazione, per esempio: odoo, il database e il server web. Una volta terminata la configurazione è possibile creare e lanciare tutti i servizi di cui abbiamo bisogno con un singolo comando.

Anche per questi strumenti l'installazione varia in base al sistema operativo usato.

### Linux

#### Docker 


Per Ubuntu:

Per prima cosa aggiungete il repository docker ufficiale:

```
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

Aggiornate l'indice dei pacchetti:

```
$ sudo apt-get update
```

Infine installate l'ultima versione di docker:

```
$ sudo apt-get install docker-ce
```

Per le altre distribuzioni e' possibile trovare le istruzioni dettagliate nella [pagina ufficiale della documentazione di Docker](https://docs.docker.com/install/linux/docker-ce/centos/)

#### Docker Compose

Per installare Compose:

Scaricate l'eseguibile con il comando:

```
sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker compose
```

Aggiungete il permesso d'esecuizione al file binario:

```
sudo chmod +x /usr/local/bin/docker compose
```

Testate l'installazione con:

```
$ docker compose --version
docker compose version 1.21.2, build 1719ceb
```

### Mac

#### Docker e Docker Compose

Per Mac è sufficiente scaricare [l'applicazione ufficiale dall'Docker Store](https://store.docker.com/editions/community/docker-ce-desktop-mac)

Per maggiori informazioni qui trovate la [pagina di documentazione specifica](https://docs.docker.com/docker-for-mac/install/#install-and-run-docker-for-mac)


### Windows

#### Docker e Docker Compose

Come per Mac è sufficiente scaricare [l'applicazione ufficiale dall'Docker Store]
(https://store.docker.com/editions/community/docker-ce-desktop-windows)

Per maggiori informazioni qui trovate la [pagina di documentazione specifica]
(https://docs.docker.com/docker-for-windows/install/#where-to-go-next)

## Continua...

Una volta terminata questa procedura siete pronti ad iniziare il corso [installando l'ambiete di sviluppo Odoo](/odoo.workshop/basics/installazione/)
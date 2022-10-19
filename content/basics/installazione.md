---
title: "Installazione"
weight: 2
date: 2018-06-27T15:33:28+02:00
---

Una volta installati gli strumenti di sviluppo come descritto nella sezione [Precorso](/odoo.workshop/basics/precorso/) possiamo procedere ad installare l'ambiente di odoo che utilizzeremo in questo corso.

## Installazione dell'ambiente di sviluppo

Scaricate l'ambiente docker che utilizzeremo dal github in una cartella sul nostro computer. Aprite un terminale e digitate:

```
$ git clone https://github.com/metadonors/odoo.docker.git
```

Entriamo nella cartella appena scaricata:

```
$ cd odoo.docker
```

Inizializziamo il database di Odoo
```
$ docker compose run odoo upgrade -i base
```

Compose comincerà a scaricare tutte le nostre dipendenze, ,a procedura può durare diversi minuti in base alla connessione a internet disponibile. 

Infine diciamo a compose di tirare su l'ambiente:

```
$ docker-compose up
```

In seguito verranno avviati i vari componenti: odoo, postgres - il database - e nginx - il server web. Al primo avvio odoo dovrà inizializzare la struttura del database (anche questa operazione potrebbe impiegare qualche minuto).

Una volta terminata, aprite il vostro browser all'indirizzo:

http://localhost

dovreste vedere la schermata di accesso di Odoo. Per entrare inserite le credenziali predefinite:

Username: **admin**
Password: **admin**

![Installazione](/odoo.workshop/screen/installazione/installazione.png?width=60pc)


## Continua...

Una volta terminata l'installazione possiamo affrontare [i concetti base](/odoo.workshop/basics/concetti/)

---
title: "Concetti Base"
date: 2018-06-27T23:17:43+02:00
weight: 3
---

Prima di cominciare è necessario affrontare un po' di teoria che sta alla base del framework Odoo. Odoo è un framework applicativo modulare, i suoi principali componenti sono i **moduli** e le **applicazioni**.

### Moduli

I moduli sono i componenti essenziali che possono essere aggiunti a Odoo. Ogni modulo può implementare nuove funzionalità oppure modificare quelle esistenti. Ogni modulo è costituito da una cartella contenente un file _\_\_manifest.py\_\__ più altri file che implementano le sue funzionalità.

### Applicazioni

Le applicazioni sono moduli di ordine maggiore, che implementano funzionalità a più alto livello o più complesse. Sono gli elementi essenziali dell'aspetto funzionale, come per esempio l'applicazione CRM o Contabilità e sono a loro volta basate sulle funzionalità di altri moduli.

Se state implementando un modulo complesso che aggiunge funzionalità specifiche a Odoo è probabile che sia un applicazione. Se state aggiungendo o modificando alcuni semplici aspetti invece è probabile che stiate sscrivendo un modulo.

Tecnicamente non c'è nessuna differenza fra i due, semplicemente le applicazioni vengono mostrate nell'elenco delle App disponibili all'utente.

### Adattare il sistema

Generalmmente per adattare Odoo alle diverse esigenze *non si modifica mai il codice esistente*. Piuttosto si creano nuovi moduli che vanno a innestarsi sugli esistenti per modificare le funzionailtà. 

In questo corso creeremo un applicazione con pochissime dipendenze, per questioni pratiche, ma nei casi reali è molto più probabile andare a lavorare sui moduli _core_ di odoo oppure su moduli terzi resi disponibili da altri sviluppatori.

### Installiamo il nostro primo modulo

Per fare un esempio, procediamo all'installazione di un modulo base. Installeremo il modulo CRM che aggiunge funzionalità utili alla gestione dei clienti.

Il modulo è già stato pacchettizzato e inserito nel codice, possiamo quindi procedere con l'installazione da interfaccia web.

Andiamo su _http://localhost_ mettiamo username e password e accediamo alla lista delle app, selezionando dal menu in alto a sinistra.

Premiamo il tasto "Install" sull'applicazione CRM e dopo qualche minuto l'applicazione sarà pronta per essere utilizzata.

### Continua

Ora possiamo andare a creare [la nostra prima applicazione](/odoo.workshop/first_app/)
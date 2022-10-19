---
title: "Scaffolding"
date: 2018-06-27T17:59:32+02:00
weight: 1
---

In questa sezione andremo a creare il nostro primo modulo. L'applicazione d'esempio che creeremo durante il corso è una classica Todo App. Per realizzarla attraverseremo le classiche fasi di sviluppo di un applicazione Odoo.

## Lo scheletro del modulo 

Odoo offre un suo meccanismo di scaffolding piuttosto primitivo per creare nuovi moduli, è possibile visualizzare il suo funzionamento lanciando questo comando dalla cartella contente l'ambiente odoo.dockerenv:

```
$ docker-compose run odoo odoo scaffold --help
```

Nella pratica ci sono altri strumenti utili per questo scopo come, [mrbob](http://mrbob.readthedocs.io/en/latest/) con i template specifici per gli [addon Odoo](https://github.com/acsone/bobtemplates.odoo).

Vi può essere utile quando affronterete il vostro primo modulo in autonomia, ma durante questo corso creeremo tutta la struttura manualmente in modo da capirne meglio i vari aspetti.

### Quindi cominciamo

Un modulo Odoo è una cartella contenente un file _\_\_manifest\_\_.py_. La cartella deve essere anche un modulo python valido, deve quindi contenere un file _\_\_init\_\_.py_.

Il nome della cartella del modulo è un nome tecnico non visibile all'utente, deve essere valido in python quindi niente deve cominciare con una lettera e sono validi solo lettere, numeri e underscore. Nel nostro caso useremo quindi il nome _todo\_app_.

All'interno della cartella _addons/todo/_ dell'ambiente docker che abbiamo scaricato, creiamo la cartella _todo\_app_ con la seguente struttura:

```
todo_app/
    __init__.py
    __manifest__.py
```

A questo punto è ora di aprire il nostro editor di testo per modificare il file _\_\_manifest\_\_.py_, dove inseriremo il seguente contenuto:

```python
{
    'name': 'Applicazione TODO',
    'description': 'Gestisci i tuoi TODO',
    'author': 'Imthe Author',
    'license': 'LGPL-3',
    'depends': ['base'],
    'application': True
}
```

Il campo _depends_ indica i moduli da cui dipende la nostra applicazione, se non sono presenti quando verra installata, Odoo li installerà automaticamente. È necessario inserirla soprattutto quando si fa riferimento a funzionalità di terze parti. 

In questo caso abbiamo usato solo alcuni dei valori definibili nel manifesto di un modulo, nei casi reali questo file è sarà più complesso. Per una spiegazione più dettagliata del suo contenuto potete consultare la pagina della [documentazione di Odoo sul file manifest](https://www.odoo.com/documentation/11.0/reference/module.html).

### I path degli addon

Nel nostro caso Odoo sarà in grado di trovare questa applicazione perchè l'abbiamo creata nella cartella addons dell'ambiente. Ma quella cartella è stata configurata per essere fra quelle dove Odoo cerca gli addon all'avvio. È possibile specificare diversi percorsi dove odoo cerca gli addons disponibili ad essere installati, fate riferimento alla variabile _addons-path_ disponibile sia nel file di confiurazione di odoo che da linea di comando.

### Installazione del modulo

A questo punto è possibile installare il modulo. Come abbiamo fatto nella sezione precedente possiamo andare nella lista delle App, scrive _todo_ nella barra di ricerca e cliccare sul pulsante installa sul nostro modulo.

Ma...il nostro modulo non c'è.

#### La modalità sviluppatore

Il nostro modulo non c'è perchè Odoo nella sua fase iniziale mette in cache tutti i moduli che sono presenti in quel momento. Per far apparire il nostro modulo dobbiamo dirgli di ricaricare la lista delle applicazioni.

Per farlo dobbiamo entrare nella **modalità sviluppatore** che sarà essenziale per tutto il nostro lavoro. 

Per attivarla bisogna andare nell'applicazione 'Settings' e premere sulla scritta 'Activate the developer mode' presente in fondo alla pagina.

Una volta fatto possiamo tornare nella lista delle App e a questo punto sarà presente il bottone 'Update App List'. Premetelo e rieffettuate la ricerca del modulo todo che dovrebbe essere ora disponibile per essere installato

![todo](/odoo.workshop/screen/primo_modulo/todo.png?width=60pc)

Cliccate su installa ed il primo passo è compiuto

### Nota bene

Sviluppare moduli è un processo iterativo e spesso dopo dei cambiamenti, in particolare sul database, sarà necessario riavviare odoo dicendogli esplicitamente di aggiornare i moduli interessati. Se la modifica invece è solo al codice python o a delle viste sarà sufficiente riaggiornare la pagina **perchè il sistema è stato lanciato in modalita di sviluppo** (con l'opzione --dev=all vedi _docker-compose.yml_). In fase di produzione qualsiasi aggiornamento comporta il riavvio del sistema

### Continua

Ora che abbiamo creato e installato la nostra prima applicazione possiamo continuare andando a modellare i nostri dati definendo [Modelli e Campi](/odoo.workshop/first_app/primo_modello/).
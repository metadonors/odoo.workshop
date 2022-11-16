---
title: "Preambolo"
date: 2018-06-28T16:39:19+02:00
weight: 1
---

Per apportare modifiche o aggiungere funzionalità ai moduli e alle applicazioni esistenti, Odoo ci mette a disposizione diversi strumenti in base a cosa e dove vogliamo intervenire.

Odoo ha dai meccanismi interni specifici che si basano sul concetto di [Ereditarietà](https://it.wikipedia.org/wiki/Ereditariet%C3%A0_(informatica)). Tramite questi meccanismi e' possibile modificare Modelli esistenti, Viste oppure Dati (come le regole di accesso).

In questo capito andremo ad aggiungere una funzionalità all'applicazione Todo creata in precedenza. Attualmente ogni utente può lavorare solo sui suoi task, perchè non aggiungere qualche funzionalità di social networking come la possibilità di condividere i task e di commentarli con diversi utenti?

Per affrontare i meccanismi di eredità implementeremo queste funzionalità in un nuovo modulo che estende quello precedentemente creato.

Questo è il nostro piano:

- Estendere il modello TodoTask aggiungendo l'utente attualmente in carico 
- Modificare la logica applicativa per far si che ogni utente lavori solo sui task di cui è incaricato
- Estendere le viste per aggiungere i campi nessari a quanto sopra
- Aggingere un wall di messaggi in fondo a ogni Task dove gli utenti possono commentare e seguire (tipo Twitter follow) l'evoluzione del task

Per procedere dobbiamo creare un nuovo modulo nella stessa cartella contenente il modulo _todo\_app_. Lo chiameremo _todo\_user_ e creeremo al suo interno il file _\_\_manifest\_\_.py_

```
    todo_user/
        __init__.py
        __manifest__.py
```

Nel file di manifesto inseriemo questo contenuto

```python
{
    'name': 'Multiuser TODO',
    'description': 'Extend Todo app to work in a multiuser environment',
    'author': 'Imthe Author',
    'license': 'LGPL-3',
    'depends': ['todo_app'],
}
```

Notate che abbiamo messo una dipendenza esplicita sul modulo _todo\_app_, questo permettera a Odoo di inizializzare questo modulo dopo aver inizializzato _todo\_app_.

Un'altra differenza è che non abbiamo esplicitato _application=True_ in quanto con questo modulo non andremo a creare una nuova applicazione ma apporteremo solo modifiche a un'altra già esistente.

Una volta fatto questo possiamo aggiornare la lista dei moduli e installare il modulo _todo\_user_ nel nostro Odoo.

A questo punto possiamo andare a vedere come si [estende il modello TodoTask](/odoo.workshop/inheritance/estendere_modelli/)


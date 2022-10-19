---
title: "I modelli"
date: 2018-06-28T10:36:46+02:00
weight: 2

---

Ora che abbiamo la nostra prima applicazione iniziamo ad aggiungere un semplice modello.

I modelli sono la M del paradigma MVC, rappresento i dati su cui la nostra applicazione lavora. Sono dei modelli i Prodotti, le Fatture, i Clienti, etc.

I modelli sono descritti da classi Python che ereditano da una classe generica del framework Odoo le loro funzionalità base. Il loro compito principale è quello di tradurre il loro schema sulle tabelle del database. Odoo si occuperè poi automaticamente di installare e aggiornare il database in fase di upgrade del modulo.

## Il nostro primo modello

Il modello che andremo a definire è un semplice Todo Task, ogni task avrà un campo testo per la descrizione e un campo booleano per segnare il task come completato.

### Creazione del modello

Tutti i modelli di un modulo risiedono all'interno di una cartella _models_ all'interno del modulo, e al suo interno aggiungiamo il file _task.py_ e il file _\_\_init\_\_.py_, necessario per far capire a Python che il contenuto di _models_ è importabile.

Avremo cosi questa struttura:

```
    todo_app/
        models/
            __init__.py
            task.py
        __init__.py
        __manifest__.py
```

All'interno del file del modello _todo\_model.py_ aggiungiamo questo contenuto:

```python
from odoo import models, fields

class TodoTask(models.Model):
    _name = 'todo.task'
    _description = 'Todo Task'

    name = fields.Char('Description', required=True)
    is_done = fields.Boolean('Done?')
    active = fields.Boolean('Active?', default=True)

```

A questo punto Odoo non sa ancora dell'esistenza della nostra cartella dei modelli, ne che e' stato definito un file _todo\_models.py_. Per rendere la nostra modifica effettiva dobbiamo aggiustare le importazioni dei file negli _\_\_init\_\_.py_

All'interno del file _todo\_app/\_\_init\_\_.py aggiungiamo:

```python
from . import models
```

All'interno del file _todo\_app/models/\_\_init\_\_.py aggiungiamo:

```python
from . import task
```

A questo punto Odoo sarà in grado di riconoscere e manipolare il modello appena creato.

### Upgrade del modulo

Ogni volta che effettuiamo una modifica ad un modello, dobbiamo dire manualmente ad odoo di aggiornare (migrare) il database, per farlo abbiamo un comando 
apposito da passare ad odoo. 

Apri una nuova shell e dalla cartella contentente il fiel docker\_compose.yml, scrivi:

```
$ docker compose run odoo upgrade todo_app
```

Che fra le tante cose che scrive, dovrebbe anche dire:

```
2018-06-28 09:46:34,017 1 INFO demo odoo.modules.registry: module todo_app: creating or updating database tables
```

### Controllare il modello

Attualmente non abbiamo ancora definito viste per il nostro modello, quindi non è possibile vedere semplicemente se le modifiche sono state apportate al sistema. Abbiamo un ottima occasione però per vedere un altro strumento di sviluppo che Odoo ci offre. Attiviamo la [modalità sviluppatore](/odoo.workshop/first_app/primo_modulo/#la-modalità-sviluppatore) e andiamo nel modulo:

**Settings -> Technical -> Database Structure -> Models** 

Cerchiamo il modello _todo.task_ e sulla lista clicchiamo sul risultato ottenuto. Se tutto è andato dovremmo vedere una schermata tipo questa:

![todo](/odoo.workshop/screen/primo_modello/modello.png?width=60pc)

Che ci conferma che il modello e i campi che abbiamo definito sono stati effettivamente creati. Come si vede, Odoo si occupa automaticamente di aggiungere altri campi, tra cui i più rilevanti sono:

- _id_ è l'identificatore unico numerico assegato a ogni istanza del nostro modello
- _create\_date_ e _create\_uid_ specificano quando e chi ha creato l'istanza
- _write\_date_ e _write\_uid_ specificano quando e chi ha modificato l'istanza

Questi campi vengono aggiunti automaticamente a tutti i modelli che verranno creati.

### Continua

Adesso che abbiamo un modello di dati su cui lavorare possiamo creare [le prime viste](/odoo.workshop/first_app/prime_viste/) che ci permettarann di interagire con il database tramite l'interfaccia web.
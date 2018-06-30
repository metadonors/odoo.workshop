---
title: "Vincoli"
date: 2018-06-28T16:52:30+02:00
weight: 5
---

Per assicurare l'integrità dei nostri dati Odoo ci fornisce due strumenti principali che possiamo applicare ai nostri modelli: **Vincoli SQL** e **Vincoli Python**.

### Vincoli SQL

I vincoli SQL sono aggunti direttamente alla definizione della tabella a livello di database e sono controllati quindi direttamente da PostgreSQL. Una volta configurati non sarà quindi possibile creare eccezioni per aggirarli.

Per configurare un vincolo sql si aggiunge un attributo _sql\_constraints_ che è un alista di tuple, in ogni tupla viene espresso l'identificatore del vincolo, il codice SQL per il vincolo e il messaggio di errore da usare.

Un classico è quello di aggiungere un vincolo di unicità a un campo, per esempio:

```python
class TodoTask(models.Model):
    _sql_constraints = [
        ('todo_task_unique_name',
         'UNIQUE(name)',
         'Il titolo del task deve essere unico')
    ]
```

Fa si che non sia più possibile aggiungere due task con lo stesso titolo.


### Vincoli Python

I vincoli Python invce possono essere utilizati per fare controlli di qualsiasi tipo utilizzando una funzione specifica. La funzione deve essere dichiarata utilizzando il decoratore _@api.constraints_ indicando la lista di campi che dovranno passare i controlli. La validazione è chiamata ogni volta che si modfificheranno alcuni o tutti i campi dichiarati.

Per esempio 

```python
from odoo.exception import ValidationError

# all'intterno della classe TodoTask 
@api.constraints('name')
def _check_name_length(self):
    for todo in self:
        if len(todo.name) < 10:
            raise ValidationError('Il titolo del task è troppo corto')
```

Controlla che il titolo di ogni task sia lungo almeno 10 caratteri.


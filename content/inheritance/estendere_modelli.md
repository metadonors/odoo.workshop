---
title: "Estendere i Modelli"
date: 2018-06-28T16:39:19+02:00
weight: 2
---

In Odoo i nuovi modelli, o le estensioni di quelli esistenti, si creano definendo delle classi Python specifiche. Il meccanismo di ereditarietà di Odoo non si base su quello delle classi Python, ma su un suo approccio interno che si basa sull'attributo _\_inherit_ specificato nella class.

In base a come viene utilizzato si possono ottenere differeneti meccanismi di ereditarietà.

Noi cominceremo con il più semplice dove la classe figlia eredita tutte le funzionalità del padre e dove noi possiamo semplicemente apportare le modifiche che ci interessano.

Per prima cosa andiamo a creare la cartella _models/_ con il file _\_\_init\_\_.py_ e il file _todo\_task.py_ che ospiterà le nostre modifiche, ottenendo questa struttura:

```
    todo_user/
        models/
            __init__.py
            todo_task.py
        __init__.py
        __manifest__.py
```

Inseriamo gli import necessari per far funzionare il codice Python mettendo all'interno del file _todo\_user/\_\_init\_\_.py_ 

```python
from . import models
```

e all'interno del file _todo\_user/models/\_\_init\_\_.py_ 

```python
from . import todo_task
```

### Aggiungere campi a un modello

A questo punto siamo pronti a modificare il file _todo\_task.py_ per estendere il nostro precedente modello. Aprimo il file e scriviamo quanto segue

```python
# -*- coding: utf-8 -*-
from odoo import models, fields, api

class TodoTask(models.Model):
    _inherit = 'todo.task'

    user_id = fields.Many2one('res.users', 'Assigned to')
    deadline_date = fields.Date('Deadline')

```

Come vedete non c'è nessun meccanismo python di eredità e anche il nome della classe è irrilevante a questi fini. Il punto chiave è l'attributo _\_inherit_ che dice ad Odoo di estendere il modello _todo.task_. Le altre linee sono normali dichiarazioni di campi.


### Modificare campi esistenti

Aggiungere campi è piuttosto semplice, ma è anche possibile apportare modifiche a campi esitenti. Per farlo è sufficiente ridichiara il campo e passare solo gli attributi che si vogliono andare a modificare.  

Per esempio per modificare il tooltip del campo name possiamo aggiungere il campo

```python
    name = fields.Char(help="Cosa bisogna fare?")
```

Per vedere se le modifiche che abbiamo apportato sono corrette dobbiamo effettuare l'upgrade del modulo _todo\_user_, ricaricare la pagina e controllare che il tooltip presente nella Form View dei Todo Task sia aggiornata con quando abbiamo scritto, come nell'immagine che segue

![label](/odoo.workshop/screen/estendere_modelli/label.png?width=60pc)

### Modificare i metodi del modello

Con l'eredità dei modelli, oltre ai campi, è posssibile anche i metodi associati. Aggiungere un nuovo metodo è semplice: è sufficiente dichiarare una nuova funzione. Se invece si vuole modificare il comportamento di un metodo esistente, si può procedere sovrascrivendo il metodo stesso ed è possibile, se necessario, invocare comunque il metodo padre con la funzione _super()_ di Python. 

{{% notice warning %}}
Cambiare gli argomenti dei metodi esistenti può essere pericoloso perchè non potete sapere chi li sta già invocando. In caso sia necessario è opportuno inizializare gli argomenti con un valore predefinito.
{{% /notice%}}

Nel nostro caso vogliamo che quando un utente invoca il metodo _do\_clear\_done_ non vengano chiusi tutti i task completati ma solo quelli assegnati all'utente stesso oppure quelli non assegnati.

Per farlo aggiungiamo il seguente metodo all'oggetto _TodoTask_ nel file _models/todo\_task.py_ 

```python
@api.multi
def do_clear_done(self):
    dones = self.search([
        ('is_done', '=', True),
        '|', ('user_id', '=', self.env.uid),
             ('user_id', '=', False)
    ])

    dones.write({'active': False})
```

Vediamo invece come estendere un metodo, senza sovrascrivere completamente la logica presente nel modello originale. Per la funzione _do\_toggle\_done_ vogliamo aggiungere un controllo in caso l'utente stia cercando di chiudere un task che non gli è assegnato. In quel caso lanciamo un errore, altrimenti procediamo come consueto.

Per ottenere questo comportamente modifichiamo il file _models/todo\_task.py_, aggiungendo neglle importazioni iniziali
```python
from odoo.exceptions import ValidationError
```

e nel corpo della classe il metodo

```python
@api.multi
def do_toggle_done(self):
    for task in self:
        if tast.user_id != self.env.user:
            raise ValidationError("Solo l'incaricato può chiudere questo task")

    return super(TodoTask, self).do_toggle_done()
```

### Continua

A questo punto per poter testare queste modifiche è necessario apportare delle modifiche alle viste di questo modello [estendendo le Viste](/odoo.workshop/inheritance/estendere_viste/)


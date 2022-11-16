---
title: "Wizard"
date: 2018-07-02T16:24:39+02:00
weight: 1
---

I wizard sono uno strumento molto utile per fornire agli utenti un interfaccia per effettuare operazioni complesse sui dati. Per esempio chiedendogli dati aggiuntivi rispetto alle operazioni che stanno facendo oppure semplicemente per avere uno strumento di validazione delle operazioni da effettuare.

## Creare un wizard

I wizard di solito risiedono nella cartella _wizards/_ all'interno dei moduli. Cominciamo a creare il nosto wizard nella cartella del modulo _task\_plus_
creando la seguente struttura

```bash
todo_plus/
    models/
        [...]
    security/
        [...]
    views/
        [...]
    # Aggiungiamo questo
    wizards/
        __init__.py
        todo_wizard.py
        todo_wizard.xml
    __init__.py
    __manifest__.py
```

All'interno del file _task\_plus/\_\_init.py\_\__ aggiungiamo 

```python
from . import wizards
```

All'interno del file _task\_plus/wizards/\_\_init.py\_\__ aggiungiamo 

```python
from . import todo_wizard
```

All'interno del file _task\_plus/\_\_manifest.py\_\__ aggiungiamo 

```python
{
    'name': 'TODO Plus',
    'description': "Maggiori funzionalita per l'app TODO",
    'author': 'Metadonors',
    'depends': ['todo_user'],
    'data': [
        'security/todo_project_acl.xml',
        'security/todo_tag_acl.xml',
        'views/todo_task.xml',
        'views/todo_project.xml',
        'wizards/todo_wizard.xml' # <--- Aggiungiamo questo
    ]
}
```

## Il modello del wizard

Un wizard visualizza una Form View all'utente, solitamente all'interno di una finestra modale. Questa vista è implementata con la stessa architattura che abbiamo visto finora con l'unica differenza che il modello del wizard erediterà dalla classe _models.TransientModel_ anzichè dalla classe _models.Models_. Questo tipo di modelli vengono salvati in database come i modelli classici con l'unica differenza che vengono automaticamente cancellati ogni giorno.

Quindi apriamo il file _task\_plus/wizards/task\_wizard.py_ e scriviamo

```python

from odoo import models, fields, api

class TodoWizard(models.TransientModel):
    _name = 'todo.wizard'
    _desctiption = 'Todo Wizard per assegnazioni massive'

    todo_ids = fields.Many2many('todo.task', string='Tasks')
    new_deadline_date = fields.Date(string='Nuova deadline')
    new_user_id = fields.Many2one('res.users', string='Nuovo Responsabile')
```

## La vista del wizard

Le viste dei wizard sono uguali alle viste classiche, con due differenze:

- Può essere inserita una sezione \<footer\> per contenere i pulsanti del wizard
- Può essere inserito un pulsante con _special="cancel"_ che si ooccupa di chiudere il wizard

Quindi apriamo il file _task\_plus/wizards/task\_wizard.xml_ e scriviamo

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record model="ir.ui.view" id="todo_task_wizard_view">
        <field name="name">todo.wizard.form</field>
        <field name="model">todo.wizard</field>
        <field name="arch" type="xml">
            <form>
                <div class="oe_right">
                    <button type="object" name="do_count_todos" string="Conta"/>
                    <button type="object" name="do_select_all" string="Seleziona tutti"/>
                </div>

                <field name="todo_ids">
                    <tree>
                        <field name="name"/>
                        <field name="user_id"/>
                        <field name="deadline_date"/>
                    </tree>
                </field>

                <group>
                    <group> <field name="new_user_id"/></group>   
                    <group> <field name="new_deadline_date"/></group>   
                </group>

                <footer>
                    <button type="object" name="do_mass_update" string="Riassegna" class="oe_highlight"
                            attrs="{'invisible':[('new_user_id', '=', False)]}"/>
                    <button special="cancel" string="Annulla"/>
                </footer>
            </form>
        </field>
    </record>

    <act_window 
        id="todo_app.action_todo_wizard" 
        name="Todo Wizard" 
        src_model="todo.task" 
        res_model="todo.wizard" 
        view_mode="form" 
        target="new" 
        multi="True"/>
</odoo>

```

```
    $ docker compose run odoo upgrade todo_user
```

ricaricare la pagine nel nostro browser e dalla lista dei todo selezioniamo qualche check box. Apparirà il menu _Action_ al cui interno ci sarà il sottomenu _Todo Wizard_. Premendo si dovrebbe aprire il nostro wizard

![form](/odoo.workshop/screen/wizards/form.png?width=60pc)



## La logica applicativa del wizard

A questo punto non ci rimane che implementare la logica dietro il nostro wizard. Escluso il tasto cancel, abbiamo tre pulsanti che devono essere programmati. Cominciamo con il tasto principale _Riassegna_

Il metodo chiamato dal pulsante è _do\_mass\_update_ e, come gli altri, dovrà essere definito nel modello

Quindi apriamo il file _task\_plus/wizards/task\_wizard.py_ e aggiungiamo in cima al file

```python
from odoo import exceptions

import logging
log = logging.getLogger(__name__)
```

nel corpo della classe invece aggiungiamo 

```python
@api.multi
def do_mass_update(self):
    self.ensure_one()

    if not self.new_user_id:
        raise exceptions.ValidationError("E' necessario specificare un utente")
    
    log.debug("Mass update on todos %s" % self.todo_ids.ids)

    vals = {}

    if self.new_deadline_date:
        vals['deadline_date'] = self.new_deadline_date
    
    vals['user_id'] = self.new_user_id.id

    self.todo_ids.write(vals)
    
```

Questo metodo solleva un errore se l'utente cerca di inviare il form senza specificare un responsabile da assegnare ai todo selezionati.

Ora vediamo come implementare il conteggio dei task aperti, aggiungiamo sempre nel corpo della modello del wizard:

```python
@api.multi
def do_count_todos(self):
    count = self.env['todo.task'].search_count([('is_done', '=', False)])
    raise exceptions.Warning("Ci sono %d todo ancora aperti" % count)
```

per l'ultimo metodo (Seleziona tutti) invece c'è un piccolo problema: ogni volta che premiamo un bottone in un wizard e la chiamata non ritorna un eccezione, Oddo chiude automaticamente la finestra popup. La soluzione a questo è qquella di restituire un'azione che dice ad odoo di riaprire il wizard, vediamo come:

```python

@api.multi
def _reopen_wizard(self):
    self.ensure_one()

    # restituiamo un dizionario rappresentate l'action che riaprira' questo wizard
    return {
        'type': 'ir.actions.act_window',
        'res_model': self._name, # questo modello
        'res_id': self.id, # l'id del wizard corrente
        'view_type': 'form',
        'view_mode': 'form',
        'target': 'new'
    }

@api.multi
def do_select_all(self):
    self.ensure_one()

    open_todos = self.env['todo.task'].search([
        ('is_done', '=', False)
    ])

    self.todo_ids = open_todos

    return self._reopen_wizard()

```


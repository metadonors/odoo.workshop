---
title: "Computed Fields"
date: 2018-06-28T16:52:30+02:00
weight: 4
---

Tutti i campi che abbiamo definito fin qui sono gestiti manualmente. L'utente entra in modalità modifica di un oggetto, aggiorna i valori e preme salva. Oddo permette però di avere altri campi il cui valore non viene definito direttamente dall'utente ma viene invece calcola attraverso una funzione.

### Computed fields

Un _computed field_ viene dichiarato esattamente come i campi normali con l'unica differenza di avere un parametro _compute_ che indica la funzione che Odoo utilizzerà per calcolarne il valore.

Immaginiamo per esempio di voler mantenere nei _TodoTask_ l'informazione relativa allora stato del progetto a cui appartengono. Gestire questo dato a mano sarebbe piuttosto oneroso, un _computed field_ invece risolve il problema egregiamente.

Per aggiunger questo campo al nostro modello dei progetti apriamo il file _models/todo\_project.py_ e aggiungiamo nel corpo della classe:

```python 
    project_state = fields.Char(string='Stato progetto', compute='_compute_project_state')

    @api.depends('project_id')
    def _compute_project_state(self):
        for todo in self:
            todo.project_state = todo.project_id.state
```

Il decoratore _@api.depends_ indica a Odoo di calcolore il valore del campo solo dopo aver modificato il valore dei campi espressi nel decoratore. Quindi in questo caso verrà ricalcolato il contatore solo quando modificheremo il valore del campo _project\_id_.

Nel file delle viste dei progetti _views/todo\_tak.xml_ modifichiamo la _Form View_ per visualizzare questo valore. Aggiungiamo nel campo _arch_:

```xml

<field name='name' position='after'>
    <field name='project_id'/>
    <field name='project_state'/>
</field>

```

Aggiorniamo il modulo

```
    $ docker-compose run odoo upgrade todo_plus
```

e ricarichiamo la pagina

![project_state](/odoo.workshop/screen/computed_fields/project_state.png?width=60pc)

Ora è presente il lo stato del progetto, se proviamo a chiudere il progetto vedremo che la stessa informazione si aggiornera anche su tutti i suoi Todo.

### Filtrare i Computed Fields

I computed field non possono essere usati come filtro sull'oggetto in cui vengono definiti a meno di non implementare esplicitamente una funzione di ricerca. Quindi a fianco al parametro _compute_, dobbiamo passargli anche un argomento _search_ specificado il nome della funzione che vogliamo utilizzare.
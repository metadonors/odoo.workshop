---
title: "Computed Fields"
date: 2018-06-28T16:52:30+02:00
weight: 4
---

Tutti i campi che abbiamo definito fin qui sono gestiti manualmente. L'utente entra in modalità modifica di un oggetto, aggiorna i valori e preme salva. Oddo permette però di avere altri campi il cui valore non viene definito direttamente dall'utente ma viene invece calcola attraverso una funzione.

### Computed fields

Un _computed field_ viene dichiarato esattamente come i campi normali con l'unica differenza di avere un parametro _compute_ che indica la funzione che Odoo utilizzerà per calcolarne il valore.

Immaginiamo per esempio di voler mantenere nei _TodoTask_ l'informazione relativa allora stato del progetto a cui appartengono. Gestire questo dato a mano sarebbe piuttosto oneroso, un _computed field_ invece risolve il problema egregiamente.

Per aggiunger questo campo al nostro modello dei progetti apriamo il file _models/task\_task.py_ e aggiungiamo nel corpo della classe:

```python 
project_state = fields.Char(string='Project state', compute='_compute_project_state')

@api.depends('project_id')
def _compute_project_state(self):
    for todo in self:
        todo.project_state = todo.project_id.state
```

Il decoratore _@api.depends_ indica a Odoo di calcolore il valore del campo solo dopo aver modificato il valore dei campi espressi nel decoratore. Quindi in questo caso verrà ricalcolato il contatore solo quando modificheremo il valore del campo _project\_id_.

Nel file delle viste dei progetti _views/task\_task.xml_ modifichiamo la _Form View_ per visualizzare questo valore. Aggiungiamo nel campo _arch_:

```xml

<field name='name' position='after'>
    <field name='project_id'/>
    <field name='project_state'/>
</field>

```

ricarichiamo la pagina

![project_state](/odoo.workshop/screen/computed_fields/project_state.png?width=60pc)

Ora è presente il lo stato del progetto, se proviamo a chiudere il progetto vedremo che la stessa informazione si aggiornera anche su tutti i suoi Todo.

### Filtrare i Computed Fields

I computed field non possono essere usati come filtro sull'oggetto in cui vengono definiti a meno di non implementare esplicitamente una funzione di ricerca. Quindi a fianco al parametro _compute_, dobbiamo passargli anche un argomento _search_ specificado il nome della funzione che vogliamo utilizzare.

Nella nostra class _TodoTask_ modifichiamo l'attributo _project\_state_

```python
project_state = fields.Char(string='Stato progetto', 
                            compute='_compute_project_state',
                            search='_search_project_state',)
```

e tra i metodi aggiungiamo:

```python
def _search_project_state(self, operator, value):
    return [('project_id', operator, value)]
```

### Scrivere i Computed Fields

È anche possibile specificare una funzione che permetta di scrivere i computed fields. Questa funzione si prende carico di normalizzare l'input per forlo arrivare al database nel modo corretto. Al pari di _compute_ e _search_, la funzione si specifica tra i parametri del campo con il nome di _inverse_:

Modifichiamo il campo aggiungendo il parametro _inverse_:

```python
project_state = fields.Char(string='Stato progetto', 
                            compute='_compute_project_state',
                            search='_search_project_state',
                            inverse='_write_project_state')
```

e tra i metodi aggiungiamo:

```python
def _write_project_state(self):
    self.project_id.state = self.project_state
```

### Salvare i Computed Fields

Un'altra possibilità è quella di comunicare ad Odoo di salvare il valore del computed field in database. Dal momento che i valori vengono salvati a database non sarà più necessario specificare le funzioni _search_ e _inverse_ perchè Odoo sarà già in grado di gestirle in autonomia.
Per salvare un computed field in database è sufficiente specificare _store=True_ tra i parametri del campo.

### Related Fields

Il computed field che abbiamo implementato è un classico: si occupa semplicemente di copiare il valore di un campo di un oggetto correlato al nostro. Questo tipo di comportamento può essere gestito in completa autonomia da Odoo, senza dover per forza esplicitare tutte le funzioni necessarie.

I _related fields_ si dichiarano nello stesso modo dei _computed field_ ma specificanto l'argomento _related_ anzichè _compute_. Il valore dell'argomento _related_ sarà una stringa in dotted-notation che identifica il campo che si vuole relazionare.

Nel nostro caso sarebbe:

```python
project_state = fields.Char(string='Stato progetto', related='project_id.state')
```

## Continua

Per terminare la nostra panoramica sui modelli, vediamo ora come possiamo [esprimere dei vincoli](/odoo.workshop/models/vincoli/) d'integrità per i nostri dati.
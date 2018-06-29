---
title: "Computed Fields"
date: 2018-06-28T16:52:30+02:00
weight: 4
---

Tutti i campi che abbiamo definito fin qui sono gestiti manualmente. L'utente entra in modalità modifica di un oggetto, aggiorna i valori e preme salva. Oddo permette però di avere altri campi il cui valore non viene definito direttamente dall'utente ma viene invece calcola attraverso una funzione.

### Computed fields

Un _computed field_ viene dichiara esattamente come i campi normali con l'unica differenza di avere un parametro _compute_ che indica la funzione che Odoo utilizzerà per calcolarne il valore.

Immaginiamo per esempio di voler aggiungere ai _TodoProject_ un contatore che indica quanti _TodoTask_ sono correlati al progetto. Gestire questo contatore a mano sarebbe piuttosto oneroso, un _computed field_ invece risolve il problema egregiamente.

Per aggiunger questo campo al nostro modello dei progetti aprima il file _models/todo\_project.py_ e aggiungiamo nel corpo della classe:

```python 
    todo_counter = fields.Integer(string='Totale Todo', compute='_compute_total_todos')

    @api.depends('todo_ids')
    def _compute_total_todos(self):
        for project in self:
            project.todo_counter = len(project.todo_ids)
```

Il decoratore _@api.depends_ indica a Odoo di calcolore il valore del campo sono dopo aver modificato il valore dei campi espressi nel decoratore. Quindi in questo caso verrà ricalcolato il contatore solo quando modificheremo il valore del campo _todo\_ids_.

E nel file delle viste dei progetti _views/todo\_project.xml_ modifichiamo la Form View per visualizzare questo valore:

```xml
<record model="ir.ui.view" id="todo_project_form_view">
    <field name="name">todo.project.form</field>
    <field name="model">todo.project</field>
    <field name="arch" type="xml">
        <form string="ToDo Project Form">
            <sheet>
                <!-- Raggruppiamo i campi su due colonne e assegnamo un identificativo ai vari gruppi -->
                <group name="group_top">
                    <group name="group_left">
                        <field name="name"/>
                        <field name="todo_counter"/> <!-- Contatore di todo -->
                    </group>
                </group>

                <field name='todo_ids'/>
            </sheet>
        </form>
    </field>
</record>
```

Aggiorniamo il modulo

```
    $ docker-compose run odoo upgrade todo_plus
```

e ricarichiamo la pagina

![todo_counter](/odoo.workshop/screen/computed_fields/todo_counter.png?width=60pc)

Ora è presente il nostro contatore di Todo, se proviamo ad aggiungerne o a toglierne vedremo che il contantore si aggiorna automaticamente.
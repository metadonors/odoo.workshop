---
title: "Estendere i Dati"
date: 2018-06-28T16:39:25+02:00
weight: 4
---

A differenza delle viste i file xml contenenti dati, come i menuitem oppure le security rules, non hanno l'elemento _arch_ e non possono quindi essere modificati utilizzanto XPath. Ma possono comunque essere modificati, rimpiazzando i valori contenuti nei loro campi.

Ogni volta che dichiariamo un _\<record id="x" model="y"\>_ Odoo effettua una _insert_ o una _update_ nel database, quindi riutilizzando gli stessi id possiamo andare ad aggiornare i dati inseriti dai moduli genitori.

Vediamo nel dettaglio cosa significa.

### Modificare i Menuitem e le Action

Per fare un esempio modifichiamo l'elemento del menu creato dalla _todo\_app_ andando a modificare la sua stringa in _I miei Todo_. Per farlo possiamo aggiungere in fondo al file _views/todo\_task.xml_

```xml
<!-- Modifica l'elemento del menu -->
<record id="todo_app.menu_todo_task" model="ir.ui.menu">
    <field name="name">I miei Todo</field>
</record>
```

Possiamo anche modificare la _action_ che abbiamo creato precedentemente. Le _action_ possono avere un parametro _context_ opzionale che va a modificare il comportamento della stessa, per esempio aggiungendogli dei valori di default che verranno utilizzati per popolare _field_ o _filters_. Quello che faremo sarà modificare la action per far si che venga visualizzata la pagina della lista Todo con il filtro _I miei Task_ preselezionato. Aggiungiamo quindi in fondo al file _views/todo\_task.xml_

```xml
<!-- Modifica la action -->
<record id="todo_app.action_todo_task" model="ir.actions.act_window">
    <field name="context">{'search_default_filter_my_tasks': True}</field>
</record>
```

Se abbiamo fatto tutto correttamente dopo un upgrade del modulo 

```
    $ docker-compose run odoo upgrade todo_user
```

dovremmo vedere:

![menuactions](/odoo.workshop/screen/estendere_dati/menuactions.png?width=60pc)


### Modficare le regole di sicurezza

La nostra applicazione precedente permette agli utenti di vedere solo i Task creati da loro stessi. Ora vogliamo che i task siano accessibili all'incaricato e a tutti i _follower_ del task. Come per i _menuitem_ e le _actions_ andremo a sovrascriere la regola precedentemente creata modificando il suo campo _domain\_force_.

Essendo una regola di sicurezza, per convenzione, la metteremo nella sottocartella _security/_ in un file che chiameremo _todo\_access\_rules.xml_

```
    todo_user/
        models/
            __init__.py
            todo_task.py
        views/
            todo_task.xml
        security/
            todo_access_rules.xml
        __init__.py
        __manifest__.py
```

che andremo successivamente ad aggiungere anche al manifesto dell'applicazione:

```python
{
    'name': 'Multiuser TODO',
    'description': 'Estende la Todo app per farla diventare Multi Utente',
    'author': 'Metadonors',
    'depends': ['todo_app'],
    'data': [
        'security/todo_access_rules.xml', # <-- Aggiungiamo questa riga
        'views/todo_task.xml'
    ]
}
```

Il contenuto del file per la regola di accesso sarà quindi il seguente:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data noupdate="1">
        <record model="ir.rule" id="todo_app.todo_task_user_rule">
            <field name="name">todo.task.user.rule</field>
            <field name="model_id" ref="model_todo_task"/>
            <field name="groups" eval="[4, ref('base.group_user')]"/>
            <field name="domain_force">
                ['|', ('user_id', 'in', (user.id, False)),
                      ('message_follower_ids', 'in', [user.partner_id.id])]
            </field>
        </record>
    </data>
</odoo>
```


### Continua

Abbiamo parlato di Follower, perchè mettevamo le basi per la prossima funzionalità: aggiungere un pizzico di social networking dando agli utenti la possibilità di commentare e seguire l'evoluzione dei nostri task. Questa è una funzionalità già presente in Odoo implementarla ci permette di vedere come [riutilizzare il codice di terzi moduli](/odoo.workshop/inheritance/aggiungere_funzionalita/)
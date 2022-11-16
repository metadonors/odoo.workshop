---
title: "Estendere le Viste"
date: 2018-06-28T16:39:23+02:00
weight: 3
---

### Un po' di teoria

Le viste Form, List e Search sono definite utilizzando la struttura XML _arch_. Per estenderle dobbiamo modificare quelle definizioni XML e farlo significa localizzare i tag XML che vogliamo cambiare e introdurre le nostre modifiche in quei punti.

Cominciamo subito con un esempio di una vista che ne estende un'altra:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record model="ir.ui.view" id="todo_task_form_view_inherited">
        <field name="name">todo.task.form</field>
        <field name="model">todo.task</field>
        <field name="inherit_id" ref="todo_app.todo_task_form_view" />
        <field name="arch" type="xml">
            <!-- ...search and modify XML tags here ... -->
        </field>
    </record>
</odoo>
```

Si nota che l'unica differenza dalla dichiarazione di una vista normale è la presenza del compo _inherit\_id_ in cui si definisce l'identificativo della vista che si vuole estendere.

Il metodo standard per indicare gli elementi XML da modificare è utilizzare un espressione [XPath](https://it.wikipedia.org/wiki/XPath) per identificare un
 elemento presente nella vista. Una volta identificato si può decidere che tipo di modifica apportare.

 Per fare un esempio, se nel nostro form del modello TodoTask volessimo aggiungere il campo _deadline\_date_ **prima** del campo _is\_done_ aggiungeremmo questa righa al nostro campo _arch_

 ```xml
 <!-- Locate is_done field with xpath and specify where to add new tags --> 
<xpath expr="//field[@name]='is_done'" position="before">
    <!-- Add field deadline_date -->
    <field name="deadline_date"/>
</xpath>
 ```

 XPath è uno strumento molto potente ed espressivo, ma spesso troppo prolisso. In questo esempio, come nella maggior parte dei casi, il nostro intento è quello di localizzare uno specifico tag _field_ all'interno della vista, quindi le nostre espressioni xpath saranno spesso molto simili. Per semplificare il lavoro, Odoo fornisce una scorciatoia per questo tipo di espressione:

 ```xml
 <field name="is_done" position="before">
    <field name="deadline_date"/>
</field>
 ```

Utilizzando xpath possiamo in compenso localizzare quasiasi altro tag all'interno della vista come _\<sheet\>_, _\<form\>_, _\<group\>_ ma anche i classsici _\<div\>_. 


L'attributo _position_ può assumere diversi valori:

- **after** aggiunge all'elemnto padre, dopo il nodo indicato
- **beforer** aggiunge all'elemnto padre, prima il nodo indicato
- **inside** aggiunge all'interno del nodo indicato (default)
- **replace** sostituisce l'elemento indicato
- **attributo** modifica gli attributi XML dell'elemento indicato

### Estendere le nostre viste

Prima di procedere dobbiamo aggiungere al nostro modulo il file XML che conterrè le definizione delle nostre viste ereditate. Come per il precedente modulo aggiungiamo la cartella _views/_ e al suo interno creiamo il file _task\_task.xml_

```
    todo_user/
        models/
            __init__.py
            todo_task.py
        views/
            todo_task.xml
        __init__.py
        __manifest__.py
```

Successivamente dobbiamo dichiarare il nuovo file aggiunto nel manifesto dell'applicazione 

```python
{
    'name': 'Multiuser TODO',
    'description': 'Estende la Todo app per farla diventare Multi Utente',
    'author': 'Metadonors',
    'license': 'LGPL-3',
    'depends': ['todo_app'],
    
    # Add this section
    'data': [
        'views/task_views.xml'
    ]
}
```

Inizializziamo il nostro file todo_task.xml con questo contenuto:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
</odoo>
```

### Estendere la Form View

Per mettere insieme quello che abbiamo detto fin'ora possiamo procedere a modificare la nostra Form View aggiungendo i campi nuovi e andando a nascondere il campo active. Aggiungiamo quindi la nostra vista ereditata dentro il tag \<odoo\> del file _task\_task.xml_:

```xml
<record model="ir.ui.view" id="todo_task_form_view_inherited">
    <field name="name">todo.task.form</field>
    <field name="model">todo.task</field>
    <field name="inherit_id" ref="todo_app.todo_task_form_view"/>
    
    <!-- ...search and modify XML tags here... -->
    <field name="arch" type="xml">
        <!-- After field 'name' add field 'user_id' -->
        <field name="name" position="after">
            <field name="user_id"/>
        </field>

        <!-- before field 'is_done' add field deadline_date -->
        <field name="is_done" position="before">
            <field name="deadline_date"/>
        </field>

        <!-- edit attributes of field 'active' to make it invisible -->
        <field name="active" positiion="attributes">
            <attribute name="invisible">1</attribute>
        </field>
    </field>
</record>
```

Una volta effettuata la modifica possiamo aggiornare il nostro modulo con

```
$ docker compose run odoo upgrade todo_user
```

ricaricare la pagine nel nostro browser e ottenere questo form

![form](/odoo.workshop/screen/estendere_viste/form.png?width=60pc)


### Estendere la List View

Nello stesso modo procediamo a modificare la List View andando ad aggiungere il campo dell'incaricato subito dopo il nome del task. Per farlo aggiungiamo un altro record al nostro file _task\_task.xml_

```xml
<record model="ir.ui.view" id="todo_task_tree_view_inherited">
    <field name="name">todo.task.tree</field>
    <field name="model">todo.task</field>
    <field name="inherit_id" ref="todo_app.todo_task_tree_view"/>
    <!-- ...search and modify XML tags here... -->
    <field name="arch" type="xml">
        <!-- After field 'name' add field 'user_id' -->
        <field name="name" position="after">
            <field name="user_id"/>
        </field>
    </field>
</record>
```

aggiorniamo il nostro modulo con

```
$ docker compose run odoo upgrade todo_user
```

ricaricare la pagine nel nostro browser e visualizziamo la lista

![list](/odoo.workshop/screen/estendere_viste/list.png?width=60pc)


### Estendere la Search View

Per finire aggiungiamo la possibilita di ricercare per utente nella nostra Search View
```xml
<record model="ir.ui.view" id="todo_task_search_view_inherited">
    <field name="name">todo.task.search</field>
    <field name="model">todo.task</field>
    <field name="inherit_id" ref="todo_app.todo_task_search_view"/>
    <!-- ...search and modify XML tags here... -->
    <field name="arch" type="xml">
        <field name="name" position="after">
            <!-- Add field 'user_id' to search-->
            <field name="user_id"/>

            <!-- Add custom filter 'My tasks' -->
            <filter 
                name="filter_my_tasks" 
                string="My tasks" 
                domain="[('user_id','=',uid)]" />
            
            <!-- Aggiungi il filtro predefinito "Task non assegnati" -->
            <filter 
                name="filter_not_assigned" 
                string="Not assigned" 
                domain="[('user_id','=', False)]" />
        </field>
    </field>
</record>

```

aggiorniamo il nostro modulo con

```
    $ docker compose run odoo upgrade todo_user
```

ricaricare la pagine nel nostro browser e controlliamo come e' cambiato il form di ricerca

![search](/odoo.workshop/screen/estendere_viste/search.png?width=60pc)


### Continua

Ora possiamo continuare andando a vedere come modificare i [dati dei moduli](/odoo.workshop/inheritance/estendere_dati/)

---
title: "Relazioni"
date: 2018-06-28T16:52:26+02:00
weight: 3
---

Abbiamo visto come lavoare con tipi di campi non relazionali, ma nelle applicazioni reali una buona parte delle strutture dati descrivono relazioni fra le entità.  L'ORM di Odoo è in grado di gestire i vari tipi di relazioni fra oggetti, mettendo a disposizione un set di funzionalità pronte all'uso.

Per avere chiaro cosa stiamo per fare, chiariamo cosa vogliamo costruire. Finora abbiamo lavorato con l'oggetto _TodoTask_. Adesso aggiungeremo il modello _TodoProject_ che raggruppa i Task, e il modello _TodoTag_ che li classifica. Ogni _TodoTask_ può appartenere a un solo progetto, c'è quindi una relazione N a 1 fra di loro. Stessi tag possono invece essere assegnati a diversi _TodoTask_, avranno quindi una relazione N a N tra di loro.

### Relazioni Many2one

Nel caso della relazione fra progetti e task abbiamo detto che esiste una relazione Molti a uno (o Uno a molti guardandola dall'altra parte). Per aggiungere questo tipo di relazione in odoo possiamo ricorrere al tipo di campo **Many2one**.

Apriamo quindi il file _models/task\_task.py_ contenente il nostro modello _TodoTask_ e aggiungiamo questo campo alla classe:

```python
project_id = fields.Many2one('todo.project', string='Project')
```

Successivamente apriamo il nostro _views/task\_task.xml_ e aggiungiamolo anche alla vista form:

```xml
<field name='name' position='after'>
    <field name='project_id'/>
</field>

```

## Relazioni One2many

Considerando invece la relazione dal punto di vista opposto, quello dei progetti, la relazione è inversa: ogni _TodoProject_ sarà legato a tanti oggetti _TodoTask_. Questo tipo di relazione in Odoo è chiamata **One2many**. Per definirla è necessario definire nella dichiarazione del campo su quale attributo dell'oggetto figlio avviene la relazione. Uno stesso oggetto potrebbe avere più relazioni One2many verso la stessa classe di oggetti.

Per aggiungere questa relazione al nostro modello dei progetti aprima il file _models/task\_project.py_ e aggiungiamo nel corpo della classe:

```python
    todo_ids = fields.One2many('todo.task', 'project_id', string='Todos')
```


{{% notice note %}}
Per convenzione su Odoo i nomi dei campi relazionali vengono sempre terminati con _\_id_ o _\_ids_. Questo per indicare velocemente agli sviluppatori se si tratta di una relazione a uno oppure a molti. Consiglio **vivamente** di continuare con questa convenzione.
{{% /notice%}}

E nel file delle viste dei progetti _views/task\_project.xml_ aggiungiamo questo record:

```xml
<record model="ir.ui.view" id="todo_project_form_view">
    <field name="name">todo.project.form</field>
    <field name="model">todo.project</field>
    <field name="arch" type="xml">
        <form string="ToDo Project Form">
            <header>
                <field name='state' widget='statusbar'/>
            </header>
            <sheet>
                <group name="group_top">
                    <group name="group_left">
                        <field name="name"/>
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
    $ docker compose run odoo upgrade todo_plus
```

e ricarichiamo la pagina

![todo_ids](/odoo.workshop/screen/relazioni/todo_ids.png?width=60pc)

Questa volta Odoo ci renderizza il campo come una lista di oggetti Todo, entrando in modalità modifica vediamo che è possibile aggiungere righe cercandole da quelle esistenti oppure crearne di nuove tramite la Form View specifica degli oggeti _TodoTask_.


## Relazioni Many2many

Le relazioni Many2Many sono molto simili alle One2Many, nel senso che ad un oggetto ne vengono legati molti e sono quindi spesso rappresentato con lo stesso widget delle One2Many. La grossa differenza però è a livello di database.

**One2many**: gli oggetti sono correlati da un id presente in una colonna della tabella figlia, nel nostro caso la colonna _project\_id_ della tabella dei todo
**Many2many**: viene creata una terza tabella composta da due colonne che fanno riferimento agli id delle due entità in relazione.

Nel nostro esempio vogliammo aggiungere una relazione fra il modello _TodoTag_ e il modello _TodoTask_. Per farlo apriamo prima il file del modello _models/task\_task.py_ e aggiungiamo il campo:

```python
tag_ids = fields.Many2many('todo.tag', string='Tag')
```

Successivamente apriamo il nostro _views/task\_task.xml_ e aggiungiamolo anche alla vista form:

```xml
<!-- All'interno del campo arch aggiungiamo  -->
<field name='user_id' position='after'>
    <field name='tag_ids'/>
</field>

```

Aggiorniamo il modulo

```
$ docker compose run odoo upgrade todo_plus
```

e ricarichiamo la pagina

![tags1](/odoo.workshop/screen/relazioni/tags1.png?width=60pc)

Il campo Tag è stato aggiunto e visualizzato come una lista di oggetti.

#### Una finezza

Odoo ha diversi widget che possono essere utilizzati per renderizzare i campi agli utenti (e anche loro possono essere aggiunti o modificati). Nel caso specifico Odoo ha un widget per gestire i tag. Per utilizzarlo bisogna modificare la dichiarazione nella vista form aggiungendo _widget="many2many\_tags"_ tra gli attributi del campo _tag\_ids_, come segue

```xml
<field name='user_id' position='after'>
    <field name='tag_ids' widget="many2many_tags"/>
</field>
```

Aggiornando la pagina otteniamo una visualizzazione del campo molto più comoda ed inerente allo stesso

![tags2](/odoo.workshop/screen/relazioni/tags2.png?width=60pc)

## Continua

Ora che abbiamo creato le nostre prima relazioni possiamo occuparci di un altro strumento molto utile che Odoo ci fornisce sui modelli, i [computed fields](/odoo.workshop/models/computed_fields/)

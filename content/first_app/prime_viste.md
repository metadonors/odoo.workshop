---
title: "Le viste"
date: 2018-06-28T10:38:52+02:00
weight: 3
---

Le viste descrivono l'interfaccia utente. Ogni vista è strutturata in un file XML, che viene utilizzato dal client web per generare le pagine HTML in grado di gestire i dati generati dal nostro backend.

Nelle viste abbiamo dei _menu item_ che ci permettono di attivare funzionalità o navigazione scatenando delle _actions_. Per esempio, il menu item **Users** processa una action chiamata anchessa **Users**, che renderizza le viste per la gestione degli utenti.

Per ogni modello sono disponibili diversi tipi di viste, le principali che utilizzeremo sono: **list view**, **form view** e **search view**.

### Creazione del MenuItem e della ActWindow

Generalemente le viste sono raggruppate nella cartella _views_ all'interno del modulo. Prima della creazione della vista vera e propria, abbiamo bisogno di un _menu item_ che ci permetta di navigare alla nostra applicazione _Todo Task_. Per farlo creiamo la cartella _views_ e al suo interno il file _task\_menu.xml_ ottenendo la seguente struttura

```
todo_app/
    models/
        __init__.py
        task.py
    views/
        task_menu.xml
    __init__.py
    __manifest__.py
```

e aggiungiamo il seguente contenuto che definisce l'oggetto menu e l'azione necessaria per navigare verso l'applicazione:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <!-- Action to open list view Todo Task -->
    <record model="ir.actions.act_window" id="action_todo_task">
		<field name="name">Todo Task</field>
		<field name="res_model">todo.task</field>
		<field name="view_mode">tree,form</field>
		<field name="domain">[]</field>
		<field name="context">{}</field>
	</record>

    <!-- Menu object used to trigger action action_todo_task -->
     <record model="ir.ui.menu" id="menu_todo_task">
        <field name="name">Todos</field>
        <field name="action" ref="action_todo_task"/>
        <field name="sequence" eval="20"/>
    </record>

</odoo>
```

Questo piccolo pezzo di codice è un file _Odoo data_. Odoo lo legge e aggiunge nel suo database queste definizioni. Nello specifico abbiamo definito:

- un elemento _\<act\_window\>_ che definice un'azione client side che apre le viste relative al modello di nome _todo.task_ con le viste _tree_ e _form_ abilitate

- un elemento _\<menuitem\>_ che definisce un menu primario che una volta cliccato scatena l'azione con id _action_todo_task_

Entrambi questi elementi hanno un attributo _id_, spesso chiamato **XML ID**, che è molto importante: è utilizzato nel modulo e all'esterno del modulo per fare riferimento a quell'oggetto specifico. Quindi, per esempio, per invocarlo, modificarlo o eliminarlo. Nel caso specifico il _\<menuitem\>_ utilizza l'id della _\<act\_window\>_ per invocarla.

{{% notice note %}}
Con viste _Tree_ in Odoo si intendo i listati. Vengono chiamati tree per ragioni storiche anche se attualmente non hanno più un comportamento da struttura ad albero se non in rari casi, come i raggruppamenti, dove permettono di visualizzare i dati su due livelli.
{{% /notice%}}


Come per i modelli, Odoo attualmente non sa dell'esistenza del file che abbiamo appena creato. Per dichiararlo dobbiamo aggiungere al file _\_\_manifest\_\_.py_ l'attributo _data_ che elenca tutte le risorse (codice Python escluso) disponibili in questo modulo.

Quindi aprimo il file _\_\_manifest\_\_.py_ e modifichiamolo di conseguenza:

```python
{
    'name': 'TODO Application',
    'description': 'Manage your Todos',
    'author': 'Imthe Author',
    'license': 'LGPL-3',
    'depends': ['base'],
    'application': True

    # Add this section
    'data': [
        'views/task_menu.xml'
    ]
}
```

{{% notice warning %}}
Ogni definizione che aggiungiamo in un file XML viene letta e aggiunta nel database da Odoo in fase di upgrade del modulo. Quando **aggiungiamo** oggetti dobbiamo effettuare un upgrade manuale del modulo per renderli disponibili. In modalita sviluppo del backend (--dev=all) le **modifiche** invece vengono lette automaticamente ed è sufficiente ricaricare la pagina. In produzione invece è **sempre** necessario effettuare l'upgrade del modulo.
{{% /notice%}}

A questo punto non ci resta che effettuare l'upgrade del modulo e ricaricare la nostra pagina.

```
$ docker compose run odoo upgrade todo_app
```

Ricarichiamo la pagina e...niente, non e' cambiato nulla. 

Il motivo per cui odoo non ci mostra il nostro nuovo modello e' legata alla gestione dei permessi: non avendo definito nessuna [politica 
di ACL](/odoo.workshop/first_app/controllo_accessi/) legata a questo nuovo oggetto, odoo lo esclude a priori

Ci occuperemo dei permessi più tardi, per proseguire attiviamo la *modalità superuser* di odoo 

![superuser](/odoo.workshop/screen/prime_viste/superuser.png?width=60pc)

Se abbiamo fatto tutto correttamente, dovremmo vedere:

![todolist](/odoo.workshop/screen/prime_viste/list.png?width=60pc)


### Creazione delle vista Form

Premendo il tasto _Create_, Odoo ci presenta un Form di default basato sulla definizione dei campi che gli abbiamo dato. Il form presentato è molto basilare, presenta semplicemente i campi definiti, nelle applicazioni reali andremo a creare un form specializzato per ogni tipo di modello.

{{% notice note %}}
Le viste _Form_ in odoo hanno un duplice utlizzo: modifica e visualizzazione dei dati. Quindi definendo una vista Form stiamo nello stesso momento creando due schermate: quella utilizzata per visualizzare i dati di un record e quella per modificarli. Rimane comunque possibile modificare questo comportamento specificando viste distinte per la visualizzazione e la modifica.
{{% /notice%}}

Per farlo dobbiamo aggiungere, dichiarandolo in un file XML Odoo data, un oggetto di tipo _ir.ui.view_. Quindi creiamo un nuovo file XML all'interno della cartella view di nome _task\_views.xml_ e otteniamo: 

```
todo_app/
    models/
        __init__.py
        task_model.py
        task_views.py
    views/
        task_menu.xml
        task_views.xml
    __init__.py
    __manifest__.py
```

All'interno di questo file aggiungiamo il seguente contenuto:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record model="ir.ui.view" id="todo_task_form_view">
        <field name="name">todo.task.form</field>
        <field name="model">todo.task</field>
        <field name="arch" type="xml">
            <form string="ToDo Task Form">
                <group>
                    <field name="name"/>
                    <field name="is_done"/>
                    <field name="active" readonly="1"/>
                </group>
            </form>
        </field>
    </record>
</odoo>
```

Questo codice aggiungne una vista con l'idendificativo _todo_task_form_view_. L'attributo model indica a Odoo che deve usare questa vista sugli oggetti di tipo  _todo.task_. L'attributo _arch_ indica il contenuto effetivo della vista e il tag _form_ specifica che la vista definita è di tipo _Form View_.

Come per il menu, dobbiamo aggiungere questo file al manifesto del modulo per far si che Odoo lo riconosca e lo carichi. Apri quindi il _\_\_manifest\_\_.py_ modficalo come segue:

```python
{
    'name': 'TODO Application',
    'description': 'Manage your Todos',
    'author': 'Imthe Author',
    'license': 'LGPL-3',
    'depends': ['base'],
    'application': True,
    'data': [
        'views/task_menu.xml',
        'views/task_views.xml', # Add this line
    ]
}
```

Una volta fatto possimo effettuare l'upgrade del modulo e aggiornare la pagina.

```
$ docker compose run odoo upgrade todo_app
```

Se tutto è andato bene otterremo una pagina di questo genere:

![todoform](/odoo.workshop/screen/prime_viste/form.png?width=60pc)

Che ci permetti di effettuare le basiche operazioni CRUD sui nostri modelli. Ma vediamo come possiamo migliorare ancora un po' la visualizzazione dei dati.

Sebbene ci sia libertà nella strutturazione delle pagine in Odoo, ci sono alcune linee guida che è utile sapere (e seguire) per rendere più semplice la vita agli utenti.

La struttura delle _Form View_ di Odoo è piuttosto standard e prende ispirazione dai fogli di carta. La pagina è suddivisa in due parti: 
- un tag _\<header\>_ che fa riferimento alla barra orizzontale in alto, dove solitamente si posizionano i bottoni di azione e gli stati dell'oggetto
- un tag _\<sheet\>_ che rappresenta la parte centrale della pagina (il foglio di carta), dove vengono rappresentati i dati.

Un altro elemento utile nei from è il tag _\<group\>_ che permette di ragguppare i campi, suddividendoli su più colonne.

Per creare queste due aree e aggiungere un paio di bottoni modifichiamo il nostro file _task\_views.xml_, aggiungendo queste due aree:

```xml

    <form string="ToDo Task Form">
        <!-- Add header, where action buttons will be placed-->
        <header>
            <!-- Add a button to edit task done state  -->
            <button 
                name="do_toggle_button" 
                type="object"
                string="Toggle Done"
                class="oe_highlight"/>
            
            <!-- Add a button to clear done tasks -->
            <button 
                name="do_clear_done" 
                type="object"
                string="Clear All Done"
                class="oe_highlight"/>
        </header>

        <!-- Add sheet section to contain the form -->
        <sheet>
            <!-- Group fields in two columns  -->
            <group name="group_top">
                <group name="group_left">
                    <field name="name"/>
                </group>
                <group name="group_right">
                    <field name="is_done"/>
                    <field name="active" readonly="1"/>
                </group>
            </group>
        </sheet>
    </form>
    
```

{{% notice note %}}
In questo caso, non avendo aggiunto record nel data file sarà sufficiente ricaricare la pagina per vedere le modifiche
{{% /notice%}}


### Creazione delle vista List

Anche le viste Tree (le list view) possono - e devono - essere personalizzate in base al modello, per farlo si va a definire un altro record nel file delle viste del modello come nel cado delle Form View, ma questa volta utilizzando il tag _\<tree\>_.

Aprimo quindi il file _task\_views.xml_ e aggiungiamo:

```xml
<record model="ir.ui.view" id="todo_task_tree_view">
    <field name="name">todo.task.tree</field>
    <field name="model">todo.task</field>
    <field name="arch" type="xml">
        <tree decoration-muted="is_done==True">
            <field name="name"/>
            <field name="is_done"/>
        </tree>
    </field>
</record>
```

Con questo codice creeremo un listato con due colonne. Abbiamo anche utilizzato una finezza, nel tag _tree_ utlizziamo l'attributo _decoration-{$name}_ della vista. Per maggiori informazioni leggi la [documentazione ufficiale di Odoo](https://www.odoo.com/documentation/15.0/developer/reference/backend/views.html) a riguardo.



### Creazione delle vista Search

Le _Search View_ sono leggermente differenti da quelle che abbiamo visto finora. La grossa differenza è che in questo caso non abbiamo una schermata dedicata alla ricerca, ma con la definizione di una _Search View_ andiamo a modificare il blocco di ricerca presente in alto a destra  nelle _Tree View_.

Per farlo aggiungiamo un altro record al file _task\_views.xml_ come di seguito:

```xml
<record model="ir.ui.view" id="todo_task_search_view">
    <field name="name">todo.task.search</field>
    <field name="model">todo.task</field>
    <field name="arch" type="xml">
        <search>
            <field name="name"/>

            <filter 
                string="Not Done"
                name="not_done"
                domain="[('is_done','=',False)]"/>
            <filter 
                string="Done"
                name="done"
                domain="[('is_done','!=',False)]"/>
        </search>
    </field>
</record>
```

Il tag _\<field\>_ definisce i campi su cui è possibile effettuare la ricerca digitando nel box testuale. I tag _\<filter\>_ aggiungono invece dei filtri predefiniti nel menu dei filtri. Vedremo successivamente qual'è il dignificato dell'attributo _domain_.


### Continua

A questo punto siamo in grado di creare, visualizzare, cancellare, listare, cercare e filtrare i dati relativi al nostro nuovo modello, non ci rimane che [aggiungere della logica applicativa](/odoo.workshop/first_app/prima_funzione/) che ci permetta di effettuare operazioni più strutturate.
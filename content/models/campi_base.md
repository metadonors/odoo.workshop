---
title: "Campi Base"
date: 2018-06-28T16:52:23+02:00
weight: 2
---

L'[ORM](https://it.wikipedia.org/wiki/Object-relational_mapping) di Odoo permette di creare modelli complessi composti da attributi basa su tipi di campi semplici. Questi campi mappano direttamente sul database e, in base al tipo, possono avere comportamenti differenti a livello di interfaccia utente. I campi di testo verranno renderizzati come semplici tag \<input\>, le date presenteranon un calendario, le selezioni multiple un tag \<select\> e così via.

### Tipi di Campi 

Ogni tipo di campo ha può accettare una serie di parametri che variano il suo comportamento. Per il dettaglio completo delle loro specifiche rimandiamo alla [documentazione ufficiale di Odoo](https://www.odoo.com/documentation/11.0/reference/orm.html#fields). 

Di seguito facciamo una panoramica dei principale di di campi non relazionali che possono essere utilizzati:

- **Char**: è il campo base per rapresentare valori testuali. Può accettare come argomento la lunghezza massima.
- **Text**: a livello di database è identico al campo Char, ma viene renderizzato come un input multilinea a livello di interfaccia utente.
- **Selection**: rappresenta un campo testo con un predertiminato insieme di valori, viene presentato all'utente con un menù a tendina.
- **Html**: a livello di database è identico al campo Char, ma all'utente viene presentato con un editor WYSWYG che traduce direttamente in HTML e il suo input viene pulito per ragioni di sicurezza prima di essere salvato.
- **Integer**: rappresenta volari numerici interi.
- **Float**: rappresenta volari numerici in virgola
- **Date e Datetime**: vengono salvati in database come stringhe nel formato YYYY-MM-DD, all'utente ddi solito sono presentati con un calendario per semplificare l'inserimento
- **Boolean**: può assumere i valori True o False e viene rappresentato con una checkbox
- **Binary**: serve a salvare file direttamente i database. Il contenuto del file viene salvato in _base64_

#### Argomenti dei tipi di campo

Quando viene dichiarato un campo è possibile, a volte necessario, passargli degli argomenti. Alcuni tipi di campo hanno argomenti specifici per cui rimandiamo alla documentazione, ma altri sono piuttosto comuni e utilizzati spesso:

- **string**: l'etichetta utilizzata di default per il campo
- **default**: il valore di default da attribuire al campo alla creazione
- **help**: una stringa di supporto all'utente che viene visualizzato come tooltip sopra il campo
- **readonly=True**: rende il campo non scrivibile da interfaccia utente, rimane comunque possibile modiificaro programmaticamente
- **required=True**: rende il campo obbligatorio sia da interfaccia che da codice (la colonna sul database diventa NOT NULL)
- **index=True**: crea un indice sulla collonna del database
- **copy=False**: il campo viene ignorato quando si utilizza la funzione di copia del framework
- **groups**: un elenco dei gruppi abilitati a vedere e utilizzare il campo

Meritano di essere citati anche:

- **deprecated=True**: crea un WARNING log sulle applicazioni che stanno ancora utilizzando questo campo
- **oldname='old_field**: si usa quando si rinomina un campo, questo permette ad Odoo di migrare i dati del database sulla nuova colonna

### Nomi di Campi riservati

Alcuni nomi di campi non vanno utilizzati perchè vengono automaticamente istanziati da Odoo:

- **id**: la chiave primaria della tabella
- **create_date**: data e ora di creazione del record
- **create_uid**: utente che ha creato il record
- **write_date**: data e ora di ultima modifica del record
- **wirte_uid**: utente che ha effettuato l'ultima modifica al record

### Nomi di Campi particolari

Altri nomi di campi vengono utilizzati dal framework per sveltire alcune operazioni, non è quindi oppurtuno utilizzarli in contesti diversi da quello per cui sono stati pensati:

- **name**: è usato di default come nome da visualizzare per il record.
- **active (tipo _Boolean_)**: viene utilizzato per effettuare un soft-delete del record. I record con active=False vengono automaticamente esclusi dalle query a meno di non specificare la codizione nel _domain_ della query stessa
- **sequence (tipo _Integer_)**: se presente viene utilizzato come indice per ordinare le righe nelle _List View_
- **state (tipo _Selection_)**: rappresenta gli stati del ciclo di vita del record e può essere usato da altri campi per decidere quando essere visibili/leggibili/obbligatori o meno.


## Nella pratica

Andiamo a modificare il nostro modello TodoTask arricchendolo con nuovi campi (e nuovi tipi di campo) così da vedere nella pratica come vengono utilizzati.

Apriamo il file _models/todo\_task.py_ del modulo _todo\_plus/_ e modifichiamo la classe di conseguenza:

```python
class TodoTask(models.Model):
    _inherit = 'todo.task'

    state = fields.Selection([
        ('draft', 'Bozza'),
        ('progress', 'In Corso'),
        ('completed', 'Completato'),
        ('canceled', 'Annullato'),
    ], string='Stato', default='draft')

    completed_date = fields.Datetime(string='Data Completamento', states={
        'completed': [('readonly', True)],
        'canceled': [('readonly', True)]
    })

    worked_hours = fields.Float(string='Ore lavorate')
    description = fields.Html(string='Descrizione')
    internal_notes = fields.Text(string='Note interne')
```

Aggiungiamo questi campi alle nostre viste, nel file _views/todo\_task.xml_

```xml
<record model="ir.ui.view" id="todo_task_form_view_inherited">
    <field name="name">todo.task.form</field>
    <field name="model">todo.task</field>
    <field name="inherit_id" ref="todo_app.todo_task_form_view"/>
    
    <!-- ...cerca e modifica i tag XML in questo punto... -->
    <field name="arch" type="xml">
        <!-- in fondo al tag header aggiungi un widget per rappresentare il campo state -->
        <header position="inside">
            <field name='state' widget='statusbar' clickable="1"/>
        </header>
    
        <!-- dopo il campo deadline_date aggiungi i campi completed_date e worked_hours -->
        <field name='deadline_date' position='after'>
            <field name='completed_date'/>
            <field name='worked_hours' />
        </field>

        <!-- dopo il group di nome 'group_top' aggiungi un nuovo group con il campo descrizione -->
        <group name="group_top" position="after">
            <group>
                <field name="description"/>
            </group>
        </group>
    </field> 
</record>
```

A questo punto sarà necessario effettuare l'upgrade del nostro modulo 

```
    $ docker-compose run odoo upgrade todo_plus
```

Ricaricare la pagine e ottenere un form con questo aspetto

![form](/odoo.workshop/screen/campi_base/form.png?width=60pc)

## Continua

Dei modelli isolati però non sono molto utili vedimamo quindi come lavorare con i [campi relazionali](/odoo.workshop/models/relazioni/)
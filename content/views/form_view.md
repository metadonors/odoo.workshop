---
title: "Form View"
date: 2018-07-02T16:17:37+02:00
weight: 3
---

## Struttura

Le applicazioni Business-oriented spesso sono costituite da insiemi di record - i prodotti in magazzino, le fatture della contabilità, etc. La maggior parte di questi tipi di dati può essere rappresentato come un documento di carta. Odoo quindi riutilizza questa astrazione per rappresentare i suoi record.

Questa è la struttura generale di un record Odoo

```xml
 <record model="ir.ui.view" id="todo_task_form_view">
    <field name="name">todo.task.form</field>
    <field name="model">todo.task</field>
    <field name="arch" type="xml">
        <form string="ToDo task Form">
            <header>
                <!-- Section for action buttons and status widget -->
            </header>
            <sheet>
                <!-- Section for main form content -->

            </sheet>
            <div class='oe_chatter'>
                <!-- History and communication  -->
                <div class="oe_chatter">
                    <field name="message_follower_ids" widget="mail_followers"/>
                    <field name="message_ids" widget="mail_thread"/>
                </div>
            </div>
        </form>
    </field>
</record>
```

Quindi come si vede ogni documento Odoo è costituito da tre aree principali: **\<header\>** dedicato al ciclo di vita del record, **\<sheet\>** per il corpo del record e infine una sezione dedicata allo **storico e alle comunicazioni** chiamato solitamente **chatter**.

### L'intestazione: \<header\>

Nell'header solitamente sono presenti due tipi di entità: pulsanti e statusbar. I pulsanti sono normali tag _\<button\>_ e solitamente l'azione principale viene evidenziata passandogli la classe css _oe\_highlight_.

La _statusbar_ invece solitamente fa riferimento all'attributo _state_ dell'oggetto. L'attributo _state_ può essere o una semplice _Selection_ di stati oppure un campo Many2one. Nel secondo caso parliamo di _Stage_ del record. In entrambi i casi però gli attributi possono essere rappresentati con il widget _statusbar_.

```xml
 <header>
    <button
        name="do_toggle_button"
        type="object"
        string="Toggle Done"
        class="oe_highlight"/>

    <button
        name="do_clear_done"
        type="object"
        string="Clear All Done"
        class="oe_highlight"/>

    <field name='state' widget='statusbar' clickable="1"/>
</header>
```

### Il Corpo: \<sheet\>

Il tag \<sheet\> è l'area principale dedicata alla presentazione dei date. È disegnata in modo da rappresentare un foglio di carta e solitamente è suddivisa in questi componenti:

- Titolo e sottotitolo in alto
- Un box di bottoni nella parte superiore (smart buttons)
- Un'area con i campi principali del docuemnto
- Una sezione organizzata in tab (notebook)

![odoo_sheet](/odoo.workshop/screen/form_view/odoo_sheet.png?width=60pc)

#### Titolo e sottotitolo

All'interno delle viste XML oltre ai tag specifici di Odoo è possibile utilizzare anche i normali tag html. Per esempio per renderizzare l'area dedicata a titolo e sottotitolo si fa ricorso a un normale \<div\> di class _oe\_title_. Per esempio

```xml
<sheet>
    <div class="oe_title">
        <label for="name" class="oe_edit_only"/>
        <h1><field name="name"/></h1>
        <h3>
            <span class="oe_read_only">By</span>
            <label for="user_id" class="oe_edit_only"/>
            <field name="user_id" class="oe_inline"/>
        </h3>
    </div>
    <!-- ... Rest of the document ...  -->
</sheet>
```

#### Gli smart buttons

La zona in alto a destra del documento è dedicato al contenitore degli smart buttons. Sono dei normali bottoni con un estetica dedicata che solitamente contegono informazioni statistiche o contatori e un rimando ai dati che rappresentano.

Per creare un contenitore per questi bottoni si può aggiungere a fianco all'eventuale _oe\_title_ un altro div di class _oe\_button\_box_

```xml
<div name="buttons" class="oe_right oe_button_box">
    <!-- Smart buttons -->
</div>
```

#### Raggruppare i campi

Il contentuno principale dei form può essere organizzato utilizzando il tag  _\<group\>_. Questo tag crea due colonne nella sua area e, di default, i field contenuti vengono rappresentati con la loro label.

Se presente un solo tag _\<group\>_ i _\<field\>_ contenuti occuperanno due colonne, una per la label e l'altra per l'input. Se invece vengono aggiunti altri tag _\<group\>_, i field e le loro label saranno distribuiti su altre due colonne.

```xml
<group name="group_top">
    <group name="group_left">
        <!-- label + field in left column -->
        <field name="name"/>
        [...]
    </group>
    <group name="group_right">
        <!-- label + field in right column -->
        <field name="deadline_date"/>
        [...]
    </group>
</group>
```

#### Il notebook

Un altro strumento per organizzare i contenuti nei nostri form e' l'elemento _\<notebook\>_ che contiene al suo interno altri elementi chiamati pagine. E' molto utile per organizzare i dati meno utilizzati in sottosezioni facilmente raggiungibili.

Un esempio di notebook:

```xml
<notebook>
    <page string="Info" name="info">
        <!-- content of first tab -->
        <field name="other_info"/>
    </page>
    <page string="Info 2" name="info2">
        <!-- content of second tab -->
    </page>
    [...]
</notebook>
```

## Componenti

Odoo fornisce una serie di componenti aggiuntivi al normale HTML, con funzionalità di più alto livello.

### Fields

I _fields_ vengono utilizzati per rappresentare e manipolare i dati dei modelli. Prendono una serie di attributi che solitamente vengono specificati nella dichiarazione del campo nel modello ma che possono essere sovrascritti nella vista:

I principali attributi dei campi:

- **name**: identifica il nome dell'attributo di riferimento
- **string**: il testo della label implicita utilizzata per il campo
- **help**: testo mostrato in un tooltip sull'etichetta del campo quando il cursore ci si ferma sopra
- **placeholder**: un suggerimento da visualizzare all'utente all'interno del campo
- **widget**: sovrascrive il tipo di wiget da utilizzare per visualizzare il campo
- **options**: un dizionario di opzioni da passare eventualmente al widget
- **nolabel=True**: disabilita la presentazione automatica della label del campo
- **invisible=True**: nasconde il campo, ma i dati relativi sono comunque caricati e disponibili nel form
- **readonly=True**: rende il campo non modificabile
- **required=True**: rende il campo obbligatorio


### Label per i Fields

Il tag _\<label\>_ può essere utilizzato per un maggior controllo sulla visualizzazione dell'etichetta del campo. Un classico è quello di far visualizzare l'etichetta solo in modalità modifica utilizzando la classe CSS _oe\_edit\_only_

```xml
<label for="name" class="oe_edit_only" />
```

### Relational Fields

Anche i campi relazionali hanno alcune opzioni specifiche che ne modificano il comportamento. Di default attraverso un campo relazionale l'utente potrà creare, e visualizzare i dati dei modelli collegati. Per disabilitare questa funzione è possibile utilizzare l'attributo _options_

```xml
<field name="project_id" options="{'no_open': True, 'no_create': True}"/>
```

Anche _context_ e _domain_ sono molto utili con i campi relazionali. Attraverso il _context_ è posibile passare valori di default alla creazione di un oggetto collegato, mentre con il _domain_ è possibile filtrare a priori i possibili valori che l'oggetto correlato può avere.

### Widget per i Fields

Ogni tipo di campo viene visualizzato nel form con un widget di default, ma è possibile comunque sovrascrivere questo comportamento utilizzando altri widget (o creandone di specifici) all'occorrenza.

Per i campi testuali, abbiamo già disponibili questi widget:

- **email** renderizza il valore del campo come un link di tipo _mailto_
- **url**  renderizza il valore del campo com un link
- **html**  renderizza il campo come html, in fase di modifica presente un editor WYSWYG

Per i campi numerici, abbiamo già disponibili questi widget:

- **handle** utilizzato di solito con i campi che indicano la posizione, renderizza un'icona che permette il drag and drop dei record nelle liste
- **float_time** renderizza un campo float come ore:minuti
- **monetary** renderizza un campo fload con il relativo valore di valuta. Si aspetta anche un opzione _currency\_id_ che indica quale attributo del modello utilizzare per indicare la valuta corrente
- **proggressbar** renderizza un float come una barra di progresso e il valore espresso in %

Per i campi relazionali e e i Selection, abbiamo già disponibili questi widget:

- **many2many_tags** renderizza il campo come una lista di label simili a bottoni
- **selection** renderizza il campo con un menu a tendina
- **radio** rederizza il campo con una lista di radio buttons
- **priority** rappresenta il campo con una lista di stelline cliccabili (solitamente i valori in questo caso sono numerici)

## Button

I tag _\<label\>_ accettano questo tipo di attributi:

- **icon** un'icona da utilizzare all'interno del bottone, le uniche disponibili sono quelle presenti nella cartella _addons/web/static/src/img/icons/_
- **string** il testo da visualizzare all'interno del bottone
- **type** indica il tipo di azione che verrà scatenata alla pressione del pulsante
    - **object** è utilizzata per invocare un metodo python
    - **action** è utilizzata per invocare una window action
- **name** identifica l'azione specifica da invocare. Sarà quindi o il nome del metodo o l'XMLID della window action
- **args** quando il _type_ è _object_, questo attributo viene usato per passare argomenti al metodo
- **context** aggiunge valori al context
- **confirm** visualizza una schermata di conferma con un messaggio il cui testo è quello assegnato a questo attributo
- **special="cancel"** usato nei wizard, si usa sul bottone che annulla l'operazione


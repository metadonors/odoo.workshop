---
title: "List e Search View"
date: 2018-07-02T16:17:53+02:00
weight: 4
---

## Le Liste

Come abbiamo vito le liste vengono rappresentate dal tipo _tree_

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

L'attributo _decoration-NAME_ permette di passare un'espressione Python che, se vera, modifica l'aspetto della riga considerata. _NAME_ puo' essere valorizzato come _bf_ o _it_ per renderizzare la riga in grassetto o corsivo, oppure può avere una classe Bootstrap per il colore del test _muted_, _info_, _danger_ o _primary_.

Altri attributi utli sull'elemento tree:

- **default_order** permette di sovrascrivere il metodo di ordinamento di default dei dati visualizzati
- **create, delete e edit** se passati con il valore _"false"_ disabilitano l'azione corrispondente nella vista
- **editable** rende le righe modificabili direttamente nella lista, i valori possibili sono _top_ o _bottom_ e indica dove verranno aggiunti i nuovi elementi


## Le ricerche

Nelle ricerche possiamo scegliere su quali campi effettuare la ricerca di default quando digitiamo nell'input in alto a sinistra, possiamo definire dei filtri predefiniti attivabili con un click e possiamo anche fornire delle regole di raggruppamento predefinite

Ecco un esempio

```xml
<record model="ir.ui.view" id="todo_task_search_view">
    <field name="name">todo.task.search</field>
    <field name="model">todo.task</field>
    <field name="arch" type="xml">
        <search>
            <!-- field that can be quickly searched in -->
            <field name="name"/>
            <field name="user_id"/>
            
            <!-- predefined filters -->
            <filter string="Not Done" domain="[('is_done','=',False)]"/>
            <filter string="Done" domain="[('is_done','!=',False)]"/>

            <separator/>
            <!-- predefined grouping -->
            <filter string="By User" name='group_user' context="{'group_by': 'user_id'}"/>
        </search>
    </field>
</record>
```

Abbiamo qui definito due campi su cui effettuare la ricerca libera _name_ e _user\_id_. Quando l'utente comincerà a digitare nel campo di ricerca verrà fuori un menu a tendina che chiederà all'utente su quale campo effettuare la ricerca.

Poi abbiamo definito due filtri predefiniti che filtrano sul valore dell'attributo _is\_done_, questi filtri sono attivabili indipendentemente uno dell'altro e se attivi entrambi vengono applicati in _OR_. Se invece fossero divisi da un \<separator\> sarebbero applicati in _AND_

Il terzo tag _fitler_ invece setta un raggruppamente modificanto il _context_ della vista. 

### Search Field

I tag _field_ delle Search View accettano questi parametri:

- **name** il nome del campo su cui cercare
- **string** la label da visualizzare all'utente
- **operator** l'operatore di confronto utilizzato per la ricerca, di default è _=_ per i numeri e _ilike_ per gli altri tipi
- **filter_domain** in alternativa all'opzione _operator_ permette di esprimere più operatori di confronto 
- **groups** permette la ricerca su un determinato campo solo a un certo insieme di gruppi di utenti

### Search Filter

Per i tag _filter_ invece sono dispoonibili queste opzioni:

- **name** il nome del filtro, non è obbligatorio ma utile perchè può essere utilizzato per modificare il filtro nelle viste figlie oppure per preselezionarlo tramite il _context_ della vista
- **string** la labbel da presentare all'utente
- **domain** l'espressionde di dominio utilizzata quando il filtro viene selezionato
- **context**  un dizionario che modifica il _context_ della vista, solitamenet utilizzato per passare il valore di _group\_by_
- **groups** permette l'utilizzo solo a  un determinato campo solo a un certo insieme di gruppi di utenti
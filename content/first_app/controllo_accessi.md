---
title: "Controllo Accessi"
date: 2018-06-28T10:40:25+02:00
weight: 5

---

Nella scorsa sezione abbiamo terminato la creazione del nostro modulo base. Al termine abbiamo notato che Odoo si lamentava dicendo che non abbiamo specificato delle regole di accesso per il modello da noi creato

```
WARNING demo odoo.modules.loading: The model todo.task has no access rules, consider adding one. E.g. access_todo_task,access_todo_task,model_todo_task,,1,0,0,0
```

In Odoo *tutti i modelli* devono avere delle regole di accesso specificate, in caso contrario solo l'utente admin potrà accedere ai dati relativi a quel modello.

Ci sono due tipi di controlli di accesso disponibili: *access control list* e *row level access control*

- *access control list* sono le regole principale e obbligatorie da definire per modello, indicano l'accesso generico in termini booleani SI/NO che i gruppi di utenti hanno rispetto ai dati del modello declinati in _lettura_, _scrittura_, _creazione_ e _cancellazione_
- *row level access control* sono le regole che a seguito delle _access control list_ vengono applicate in termini di riga. Per esempio: tutti i dipendenti possono creare oggetti Todo ma possono visualizzare e utilizzare sono quelli creati da loro.


### Access Control List

Per farci un'idea di come sono strutturate queste regole, entriamo in [modalità sviluppo](http://localhost:1313/odoo.workshop/first_app/primo_modulo/#la-modalità-sviluppatore) e andiamo in:

**Settings -> Technical -> Security -> Access Control List**

Ci troveremo di fronte a una pagina come questa:

![acl](/odoo.workshop/screen/controllo_accessi/acl.png?width=60pc)

Quindi quello che dovremo fare è aggiungere un Odoo data file, come per le viste, contenente le informazioni che voglioamo caricare in questa tabella relative ai nostri modelli.

Solitamente le regole di accesso vengono inserite nella cartella _security_ all'interno del modulo in cui stiamo lavorando. Quindi creaiamola e al suo interno aggiungiamo un file _todo\_model\_acl.xml_ ottenendo questa struttura

```
todo_app/
    models/
        __init__.py
        todo_model.py
        todo_views.py
    views/
        todo_menu.xml
        todo_views.xml
    security/
        todo_model_acl.xml
    __init__.py
    __manifest__.py
```

All'interno del file _todo\_model\_acl.xml_ andiamo poi a definire i dati che regoleranno l'acl in questione nel seguente modo

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record model="ir.model.access" id="todo_task_user_access">
            <field name="name">todo.task.user</field>
            <field name="model_id" ref="model_todo_task"/>
            <field name="group_id" ref="base.group_user"/>
            <field name="perm_read" eval="1"/>
            <field name="perm_create" eval="1"/>
            <field name="perm_write" eval="1"/>
            <field name="perm_unlink" eval="1"/>
        </record>
    </data>
</odoo>
```

Questa codice aggiunge una regola di acl con _id_ todo_task_user_access che viene applicata al modello _todo.task_ sul gruppo _base.group.user_ dando tutti i permessi di lettura, scrittura, creazione e cancellazione. Il gruppo _base.group.user_ è un gruppo generico di Odoo a cui fanno parte tutti gli utenti del sistema.

{{% notice note %}}
In questo caso abbiamo aggiunto al tag _data_ il valore _noupdate_. Questo dice ad Odoo di creare la riga nel database al primo upgrade, ma di non aggiornarla durante un successivo upgrade se già presente. Questo perchè questo tipo di dati può essere modificato dall'amministratore direttamente da interfaccia web in produzione e non vogliamo che vengano resettate queste modifiche ad un successivo upgrade del modulo.
{{% /notice %}}


Come per le viste dobbiamo poi specificare nel file _\_\_manifest\_\_.py_ l'esistenza di questa regola, apriamo quindi il file manifesto e modifichiamolo come segue:

```python
{
    'name': 'Applicazione TODO',
    'description': 'Gestisci i tuoi TODO',
    'author': 'Fabrizio Arzeni',
    'depends': ['base'],
    'application': True,
    'data': [
        'security/todo_model_acl.xml', # <-- Aggiungiamo questa riga
        'views/todo_menu.xml',
        'views/todo_views.xml',
    ]
}
```

Avendo aggiunto una nuova risorsa dobbiamo ricordarci di fare l'upgrade del modulo lanciando da terminale:

```
$ docker-compose run odoo upgrade todo_app
```

A questo punto il messaggio di WARNING dovrebbe essere scomparso e tutti i nostri utenti hanno ora accesso al modello del nostro modulo.

### Row-level access rule

Immagina di desiderare che gli utenti che usano il nostro modulo possano accedere solo ai dati creati da loro stessi. Le regole di accesso non sono sufficienti per specificare questa logica, perchè anche in questo caso tutti avrebbero gli stessi permessi di accesso, cambierebbe solo su **quali** dati possono applicarle.

In questo caso si utilizzano le regole di accesso per riga, che selezionano tramite un _domain_ i record ai quali applicare le regole di accesso generali. Nel nostro caso vorremo che gli utenti abbiano accesso solo a quei record che hanno il campo create_user uguale a quello dell'utente che sta utilizzando il sofware ma possono essere espresse anche regole ben più complesse.

Come per le _Access Control List_ le regole di riga vengono salvate in una specifica tabella di Odoo che può essere visualizzata andando nella sezione

**Settings -> Technical -> Security -> Record Rules**

![rules](/odoo.workshop/screen/controllo_accessi/rules.png?width=60pc)

e sempre come le ACL possiamo creare delle regole aggiungendole al file di security che abbiamo creato in precedenza. Quindi apriamo il file _security/todo\_model\_acl.xml e aggiungiamo dopo il record creato in precedenza

```xml
<record model="ir.model.access" id="todo_task_user_rule">
    <field name="name">todo.task.user.rule</field>
    <field name="model_id" ref="model_todo_task"/>
    <field name="domain_force">
        [('create_uid', '=', user.id)]
    </field>
    <field name="groups" eval="[4, ref('base.group_user')]"/>
</record>
```

Se tutto è andato bene la regola sarà aggiunta.

### Continua

Il nostro modulo sembra essere ad un buon punto ormai da meritare possiamo quindi andare a scoprire [come si modificano le applicazioni esistenti](/odoo.workshop/inheritance/)
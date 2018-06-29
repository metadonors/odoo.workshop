---
title: "Aggiungere funzionalità"
date: 2018-06-28T16:39:25+02:00
weight: 5
---

Finora abbiamo visto come estendere applicazioni esistenti aggiungendo funzionalità create da noi. Ma Odoo permette di fare molto di più: possiamo aggiungere funzionalità implementate di moduli scritti da altre persone.

Nel nostro caso vogliamo aggiugere la possibilità per gli utenti di commentare i task e di poterli seguire, alla maniera di Twitter. Questa funzionalità in Odoo è implementata nel modulo _mail_, più peccificatamente nel modello _mail.thread_ di quel modulo.

Per aggiungerlo al nostro modello _todo.task_ dobbiamo procede in questo modo:

- Aggiungere il modulo _mail_ alle nostre dipendenze
- Aggiungere la classe _mail.thred_ a quelle che eredita il nostro modello _todo.task_
- Aggiungere i campi necessari nella nostra vista
- Modficare le regole di accesso per questa funzionalità (lo abbiamo già fatto nella [sezione precedente](/odoo.workshop/inheritance/estendere_dati/#modficare-le-regole-di-sicurezza/))


### Modificare le dipendenze

Apriamo il nostro file di manfiesto e modifichiamolo come segue:

```
{
    'name': 'Multiuser TODO',
    'description': 'Estende la Todo app per farla diventare Multi Utente',
    'author': 'Metadonors',
    'depends': [
        'todo_app',
        'mail' # <-- Aggiungiamo la dipendenza
    ],
    'data': [
        'security/todo_access_rules.xml', # <-- Aggiungiamo questa riga
        'views/todo_task.xml'
    ]
}
```

### Aggiungere la classe al modello

Per aggiungere la classe al modello procediamo nella stessa maniera con cui abbiamo ereditato il modello esteso dal modello base. Andiamo ad aggiungere all'attributo _\_inherit_ il modello _mail.thread_

Apriamo il nostro file _models/todo\_task.py_ 

```python
class TodoTask(models.Model):
    _name = 'todo.task'
    _inherit = ['todo.task', 'mail.thread']
```

Con questa semplice modifica il nostro modello thread acquisterà tutta la logica che gli serve per far funzionare la messaggistica. C'è anche da notare che mettendo due dipendenze siamo obbligati anche a specificare quale deve essere il nome del modello principale da cui ereditiamo, per questo aggiungiamo ancche l'attributo _\_name_.

{{% notice note %}}
Il modello _mail.thread_ è un **Abstract Class**, che significa che non ha una tabella sua di riferimento. Può essere ssolo utiilizzato in altri modelli e va a estendere le loro funzionalità. Quindi, nella pratica, aggiungendolo andrà a modificare direttamente la tabella originale e aggiungerà al modello i metodi necessari alle sue funzionalità. Per maggiori infomazioni consultate la [documentazione ufficiale di Odoo](https://www.odoo.com/documentation/11.0/reference/orm.html#model-reference).
{{% /notice%}}


### Aggiungere i campi necessari alla vista

A questo punto non ci rimane che aggiungere il widget delle funzionalità mail al fondo della nostra Form View del modello. Apriamo il file _views/todo\_task.xml_ e aggiungiamo nell'attributo _arch_ questo codice XML

```xml
<sheet position="after">
    <div class="oe_chatter">
        <field name="message_follower_ids" widget="mail_followers"/>
        <field name="message_ids" widget="mail_thread"/>
    </div>
</sheet>
```

Se ci fate caso i due campi _message\_followers\_ids_ e _message\_ids_ non sono stati implementati esplicitamente da noi ma vengono ereditati dalla classe _mail.thread_.

A questo punto non ci resta che effettuare l'upgrade del nostro modello

```
    $ docker-compose run odoo upgrade todo_user
```

ricaricare la pagina e cominciare ad aggiungere commenti

![chatter](/odoo.workshop/screen/aggiungere_funzionalita/chatter.png?width=60pc)

### Continua

Il nostro upgrade del modulo base è finito, a questo punto possiamo andare a guardare più nel dettaglio cosa ci offre il framework Odoo. Cominciamo ad [analizzare i Modelli](/odoo.workshop/models/)
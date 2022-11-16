---
title: "La logica"
date: 2018-06-28T10:38:58+02:00
weight: 4
---

A questo punto è arrivato il momento di aggiungere della logica applicativa al nostro modulo. Per farlo possiamo sfruttare i bottoni che abbiamo aggiunto nella sezione precedente [relativa alle viste](/odoo.workshop/first_app/prime_viste/).


### Aggiungere la logica applicativa

Nella precedente sezione abbiamo aggiunto due bottoni per invocare delle funzioni Python sul nostro backend. Riporto qui il codice interessato per comodità.

```xml
    <button 
        name="do_toggle_button" 
        type="object"
        string="Toggle Done"
        class="oe_highlight"/>
```

In questo tag stiamo definendo alcuni attributi, vediamoli nel dettaglio:

- **string** è la stringa che verrà visualizzata all'utente all'interno del bottone
- **class** è la classe CSS che verrà applicata al bottone (in questo caso ne applichiamo una predefinita in Odoo)
- **type** è il tipo di azione che deve essere scatenata quando il bottone viene premuto, può essere valorizzato come 
    - _object_ per invocare un metodo del modello
    - _action_ per invocare un'azione generica (come quella che abbiamo utilizzato per accedere all'applicazione)
- **name** nel caso in cui _type=object_ name rappresenta il nome del metodo che verrà invocato, in questo caso _do\_toggle\_button_

Per altre informazioni fai riferimento alla [documentazione ufficiale di Odoo](https://www.odoo.com/documentation/11.0/reference/views.html#reference-views-list-button)

Quindi avendo definito un bottone con _type=object_ e _name=do\_toggle\_button_ non ci rimane che aggiungere questo metodo alla classe Python che rappresenta il modello. Apriamo quindi il file _models/task\_model.py_ e aggiungiamo un metodo alla classe:

```python
# In TodoTask class add this method
def do_toggle_button(self):
    for todo in self:
        todo.is_done = not todo.is_done
```

Poche righe, ma tanti concetti. 

Odoo è un software scritto con una logica **API first**, che significa che il codice che scrivete nel backend (in Python) è completamente slegato da quello che avviene nel frontend (che invece è scritto in Javascript). Ogni funzione che volete esporre al frontend deve essere esposta come un API JSONRPC (un protocollo RPC di qui potete trovare maggiori infomazioni [su internet](https://en.wikipedia.org/wiki/JSON-RPC)).

Ogni volta che viene dichiarato un metodo su un modello quel metodo puo essere eseguito su uno o più record alla volta tramite api. Quindi potete invocare questo metodo su un solo record todo in una chiamata come su 100 o 1000.

Se non si vuole esporre il metodo tramite api è sufficiente utilizzare il prefisso _ sul nome del medoto.

Il _self_ dell'oggetto rappresenta non un istanza sola, ma un [RecordSet](https://www.odoo.com/documentation/15.0/developer/reference/backend/orm.html#recordsets) che è una struttura di odoo che raggruppa oggetti dello stesso tipo e ha il principale scopo di eseguire operazioni in batch ottimizzate su molti oggetti. Quindi ciclando sul _self_ stiamo andando a lavorare su ogni record presente su cui è stata invocata la chiamata.

Per l'altro bottone invece:

```xml
    <button 
        name="do_clear_done" 
        type="object"
        string="Clear All Done"
        class="oe_highlight"/>
```

Vediamo che abbiamo dichiarato che vogliamo invocare il metodo _do\_clear\_done_ che andiamo a implementare nel nostro _models/task\_models.py_ come segue

```python
      def do_clear_done(self):
        dones = self.search([("is_done", "=", True)])
        dones.write({"active": False})
```

Con il metodo _search_ andriamo a cercare i _todo_ che ci interessano passandogli un _domain_ per la ricerca e infine aggiorniamo il loro stato come inattivo.

I _domain_ sono il principale metodo per effettuare ricerche nel database usando il framework di Odoo. Li vediamo più nel dettaglio successivamente.

A questo punto possiamo aggiornare le viste Odoo e provare le nostre nuove funzioni

```
$ docker compose run odoo upgrade todo_app
```


### Continua

Con quest'ultimo passaggio abbiamo terminato la nostra panoramica delle funzionalità base di un modulo Odoo. Rimane una piccola macchia che ci permette però di andare a guardare un ultimo aspetto riguardante i nostri modelli.

Se ci fate caso, avviando il server di odoo, viene visualizzato un WARNING di questo tipo:

```
WARNING odoo odoo.modules.loading: The models ['todo.task'] have no access rules in module todo_app, consider adding some, like:
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
```

La ragione di questo avviso è legata al meccanismo delle regole di accesso ai dati di Odoo, vediamo come funzionano e come risolvere nella [prossima sezione](/odoo.workshop/first_app/controllo_accessi/).

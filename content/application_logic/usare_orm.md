---
title: "Usare l'ORM"
date: 2018-07-02T16:24:21+02:00
weight: 2
---

Fino a questo punto abbiamo utilizzato l'ORM di Odoo senza scendere nel dettaglio su come funziona. Ora andiamo a vedere quali sono i suoi principali componenti

### Decoratori

Abbiamo notato che a molti metodi dei modelli viene applicato un decoratore come _@api.multi_. Questi decorati hanno lo scopo di istruire il backend su come gestire i metodi rispetto alle API esposte.

Il decoratore **@api.model** invece si utilizza per decorare metodi statici in cui il _self_ non fa rifermento a nessuna entità in particolare. Per coerenza _self_ farà sempre riferimento a un oggetto di tipo recordset ma il suo contenuto diventa irrilevante. Questo tipo di metodi non possono essere richiamati dalle API e quindi non possono essere usati sui bottoni nell'interfaccia utente.

Altri decoratori con scopi più specifici:

- **@api.depends(campo1, campo2, ...)** è utilizzato dai computed fields per identificare le dipendenze in base a cui scatenare il ricalcolo del valore del campo.
- **@api.constraints(campo1, campo2, ...)** è utilizzato per la validazione, la funzione viene invocata ogni volta che si cerca di modificare il valore del campo.
- **@api.onchange(campo1, campo2, ...)** è richiamata ogni volta che nella UI vengono modificati i campi elencati nel decoratore. è utile per aggiornare al volo i form con valori dipendendi da altri campi.

## La shell di odoo


Lanciando il seguente comando, odoo ci presenta un linea di comando interattiva dove possiamo accedere a tutto l'ambiente di odoo. E' molto comoda per effettuare dei test:

```
$ docker compose run odoo shell

In [1]: self
Out[1]: res.users(1,)

In [2]: self._name
Out[2]: 'res.users'

In [3]: self.name
Out[3]: 'Administrator'

```


## Recordset

I metodi principali utilizzabili nei modelli sono quelli disponibili sulla [documentazione ufficiale](https://www.odoo.com/documentation/11.0/reference/orm.html#model-reference)

### Interrogare i modelli

Attraverso la variabile _self_ possiamo accedere solamente ai metodi del modello che stiamo attualmente utilizzando. Ma ogni modello ha un variabile _env_, accessibile tramite _self.env_ che ci permette di avere un riferimento a qualsiasi modello installato sul sistema. Per esempio _self.env['res.parner']_ restituisce un riferimento al modello dei Partner permettendoci quindi di utilizzare metodi come _search_ o _browse_ su quel determinato set di dati.

Il metodo _search()_ accetta come paramentro un _domain_ e restituisce un recordset contenente le righe che rispettano le condizioni del dominio. Passando un _domain_ vuoto ([]) si ottengono tutte le righe presenti. Gli altri paramentri accettati da _search()_ sono:

- **order** e' la stringa utilizzata nella clausula _ORDER BY_ della query SQL
- **limit** il numero massimo di elementi da restituire nella query
- **offset** ingnora i primi n risultati. Combinato con _limit_ serve a gestire la paginazione degli elementi

A volte serve solo contare gli elemneti in un determinato _domain_, in quei casi è possibile utilizzare la funzione _search\_count()_ che accetta gli stessi parametri della _search()_ ma restituisce un intero (e ha un impatto infinitesimale in termini di performance rispetto alla classica _search())_

Il metodo _browse()_ prende una lista di ID oppure un singolo ID e ritorna un recordset contenente i record trovati. E' molto più performante della _search_ ma richiede la conoscenza degli ID interessati.

Alcuni Esempi:

```python
In [2]: self.env['res.partner'].search([('name','ilike','ad')])
Out[2]: res.partner(3,)

In [4]: p = self.env['res.partner'].browse(3)
In [5]: p.name
Out[5]: 'Administrator'

```

### Operazioni sui recordset

I Recordset supportano diverse operazioni. Possiamo per esempio controllare se un elemento è contenuto in un recordset oppure no.

Considerando _x_ un singleton e _test\_recordset_ un insieme di elementi possiamo scrivere

- _x in test\_recordset_
- _x not in test\_recordset_

Sono inoltre dissponibili le seguenti proprietà:

- _test\_recordset.ids_ restituisce una lista di ID relativa agli elementi contenuti
- _test\_recordset.ensure\_one()_ controlla che il recordset sia un singleton altrimenti lancia un eccezione di tipo _ValueError_
- _test\_recordset.filtered(func)_ ritorna un recordset filtrato secondo la funzione _func_
- _test\_recordset.mapped(func)_ ritorna una lista di valori mappati secondo la funzione _func_
- _test\_recordset.sorted(func)_ ritorna un recordset ordinato secondo la funzione _func_

### Manipolazione dei recordset

Per aggiungere, togliere o sostituire elementi dai recordset ci sono una serie di operatori che ci possono aiutare. I recordset di per sè sono immutabili ma attraverso questi operatori è possibile generare nuovi recordset modificati partendo da quelli esistenti

Gli operatori di manipolazione sono:

- _rs1 | rs2_ restituisce l'*unione* dei due recordset, il risultato conterrà tutti gli elementi di entrambi gli insiemi (senza doppioni)
- _rs1 + rs2_ restituisce la *somma* dei due recordset, il risultato conterrà la concatenazione degli elementi di entrambi gli insiemi (possono quindi esserci dei doppioni)
- _rs1 & rs2_ restituisce l'*intersezione* dei due recordset, il risultato conterrà solo gli elementi presenti in entrambi gli insiemi
- _rs1 - rs2_ restituisce la *differenza* dei due recordset, il risultato conterrà gli elementi presenti in _rs1_ ma non presenti in _rs2_

È inoltre possibile accedere agli elementi dei recordset attverso gli operatori di list Python, queste sono quindi espressioni valide:

- _rs[0]_ il primo elemento del recordset
- _rs[-1]_ l'ultimo elemento del recordset
- _rs[1:]_ restituisce una copia del recordset senza il primo elemento

Altri operatori:

- _rs\_ids |= element\_id_ aggiunge _element\_id_ al recordset _rs\_ids_ se non presente
- _rs\_ids -= element\_id_ rimuove _element\_id_ al recordset _rs\_ids_ se presente

### Query SQL

E' sempre possibile accedere al databse direttametne eseguendo query SQL personalizzate. Nella varibile _self.env.cr_ è disponibile un cursore legato all'attuale connessione al db che possiamo utilizzare proprio a questo scopo.

Per effetturare una query utilizziamo il metodo _execute_ successivamente dobbiamo invocare un'altra funzione per ottenerne gli eventuali risultati:
- **fetchall()** restituisce una lista di tuple rappresentanti le righe
- **dictfetchall()** restituisce una lista di dizionari rappresentanti le righe con il nome della colonna utilizzato come chiave

```python
In [1]: self.env.cr.execute("SELECT id, login FROM res_users WHERE login='%s'" % 'admin')

In [2]: self.env.cr.dictfetchall()
Out[2]: [{'id': 1, 'login': 'admin'}]
```

## Campi Relazionali

I campi relazionali possono essere utilizzati con la notazione puntata, esempio:

```python
In [14]: self
Out[14]: res.users(1,)

In [15]: self.company_id
Out[15]: res.company(1,)

In [16]: self.company_id.name
Out[16]: 'My Company'

In [17]: self.company_id.currency_id
Out[17]: res.currency(1,)

In [18]: self.company_id.currency_id.name
Out[18]: 'EUR'
```

Quando dobbiamo scrivere un campo *Many2one* dobbiamo ricordarci di passare solo l'id dell'oggetto e non il singleton corrispondente, quindi e' corretto

```python
# CORRETTO
In [17]: self.write({'user_id': self.env.user.id})
```

ma darebbe invece errore

```python
# SBAGLIATO
In [17]: self.write({'user_id': self.env.user})
```

Quando invece dobbiamo scrivere i campi *Many2many* oppure *One2many* esiste una sintassi specifica per farlo. Il valore che dobbiamo passare nella write è una tupla contenente tre valori i cui possibili valori sono:

- _(0, \_, values)_ aggiungi un nuovo record creato dal dizionario _values_
- _(1, id, values)_ aggiorna gli elementi correlati esistenti con i valori passati nel dizionario _values_ (non può essere usato con il metodo _create()_)
- _(2, id, \_)_ rimuovi la relazione con l'oggetto idenficato da _id_  e poi cancella l'oggetto rimosso dal db
- _(3, id, \_)_ rimuovi la relazione con l'oggetto idenficato da _id_  e ma non rimuovere l'oggetto rimosso dal db (solo _Many2many_ e non durante la _create()_)
- _(4, id, \_)_ aggiungi l'oggetto esistente con id = _id_  (solo _Many2many_)
- _(5, \_, \_)_ rimuovi tutti gli oggetti dalla relazione (solo _Many2many_)
- _(6, \_, ids)_ sostituisci tutti gli elementi dalla relazione con quelli identificati dagli id _ids_ (equivalente di chimare prima il comando 5 e poi il 4 su tutti gli _ids_)


---
title: "Operazioni"
date: 2018-07-02T12:16:04+02:00
weight: 2

---

La [documentazione ufficiale](https://www.odoo.com/documentation/11.0/webservices/odoo.html) ci illustra come procedere ad interrogare le API utilizzando direttamente le librerie XMLRPC disponibili nei vari linguaggi. In questa sezione invece analizziamo come interfacciarci alla nostra applicazione tramite una libreria di più altro livello, che semplifica e ottimizza le nostre chiamate.

## Login

Per effettuare qualsiasi tipo di operazione tramite API è necessario per prima cosa autenticarsi. L'autenticazione tramite API avviene utilizzando tre informazioni: username, password e nome del database dei riferimento.

Abbiamo ovviamente bisogno di conoscere anche indirizzo e porta dove gira il nostro Odoo.

Per autenticarsi apriamo una shell interattiva con

```
$ docker compose run odoo ipython
```

e al suo interno scriviamo

```python
In [1]: import odoorpc

In [2]: odoo = odoorpc.ODOO('odoo',port=8069)

In [3]: odoo.login('demo','admin','admin')

In [4]: user = odoo.env.user

In [5]: print(user.name)
Administrator

In [6]: print(user.company_id.name)
My Company
```

Nel nostro caso specifico l'installazione risponde all'indirizzo _odoo_ che abbiamo configurato nel file _docker\_compose.yml_. Nei casi reali corrisponderà all'URL della vostra installazione

## Leggere da API

Per le operazioni di lettura possiamo utilizzare diverse funzioni, disponibili anche lato backend: **browse**, **search**, **search_read**. 

- **browser**: accetta una lista di id di entità che verranno lette tutte assieme e restituisce degli oggetti rappresentanti le risorse
- **search**: accetta un dominio su cui effettuare una ricerca e restituisce gli id degli oggetti trovati (è quindi poi necessario effetturare una chiamata _browse_ per ottenerne il contenuto)
- **search_read**: accetta un dominio e un elenco di attributi e restituisce dei dizionari contenenti solo i valori richiesti

Le due funzioni di ricerca accettano anche i parametri _offset_ e _limit_ su cui si può impostare la paginazione.

Dal momento che le richieste API viaggiano su chiamate di rete che sono notoriamenet lente, la funzione _search\_read_ acquista una notevole importanza perchè ci permette di ottimizzare i payload che dovranno poi essere scambiati fra gli endpoint, facendoci restituire solo il necessario.


### Browse

Per effettuare una chiamata _browse_ ed accedere alle risorse tramite il loro id, procediamo in questo modo

```python 
In [10]: todo = odoo.env['todo.task'].browse(3)

In [11]: todo.name
Out[11]: 'WorkShop su Odoo'
```

### Search

Per effettuare una chiamata _browse_ ed accedere alle risorse tramite il loro id, procediamo in questo modo

```python 
In [12]: todos = odoo.env['todo.task'].search([('name','ilike','odoo')])

In [13]: todos
Out[13]: [3]
```

### Search_Read

Per effettuare una chiamata _browse_ ed accedere alle risorse tramite il loro id, procediamo in questo modo

```python 
In [15]: todos = odoo.env['todo.task'].search_read([('name','ilike','odoo')], ['id', 'name','description'])

In [16]: todos
Out[16]: 
[{'description': '<p><b>Un interessante descrizione</b><br></p>',
  'id': 3,
  'name': 'WorkShop su Odoo'}]
```

## Creazione di record

Per creare nuovi oggetti possiamo ricorrere alla funzione _create_

```python 
In [15]: todo_id = odoo.env['todo.task'].create({'name': 'Test da API', 'description': 'Sono un test creato da API'})

In [16]: todo_id
Out[16]: 7
```

## Modifica dei record

Per modificare oggetti esistenti ricorrere alla funzione _write_, su un oggetto resistuito da _browse_

```python
In [20]: todo = odoo.env['todo.task'].browse(3)

In [21]: todo.write({'name': 'Nuovo titolo'})
Out[21]: True

```

## Cancellazione dei record

Per cancellare oggetti esistenti ricorrere alla funzione _unlink_

```python
In [27]: odoo.env['todo.task'].unlink(3)
Out[27]: True
```

Oppure con un approccio Active Record

```python
In [20]: todo = odoo.env['todo.task'].browse(3)

In [21]: todo.unlink()
Out[21]: True

```

## Altre funzioni

Ogni volta che dichiariamo un metodo con il decoratore _@api.multi_ o _@api.model_ quello che stiamo facendo è effettivamente dichiarare un nuovo endpoint API su quel modello. Come unica restrizione di sicurezza Odoo non permette a client diversi da il proprio client web di chiamare le funzioni che iniziano con **"_"**.

Per esempio:

```python
@api.model
def io_posso_essere_chiamata_da_client_esterni(self):
    return "OK"

@api.model
def _io_non_posso_essere_chiamata_da_client_esterni(self):
    return "KO"
```

Permette di effetturare una chiamata API come questa

```python
In [20]: todo = odoo.env['todo.task'].browse(3)

In [21]: todo.io_posso_essere_chiamata_da_client_esterni()
Out[21]: "OK"

```
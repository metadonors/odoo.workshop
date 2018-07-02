---
title: "Context e Domain"
date: 2018-07-02T16:17:28+02:00
weight: 2
---

Più volte durante il corso siamo inciampati su _context_ e _domain_. Abbiamo visto che possono essere legati alle azioni, ma anche usati durante le interrogazioni del database. Vediamo più nel dettaglio che cosa sono.

## Context

Il _context_ è un dizionario Python contenente dati che vengono mantenuti durante la sessione e che possono essere utilizzati sia dal client per renderizzare diversi componente della UI, sia dal server per modificare la logica applicativa

Nel client ci sono sia informazioni che riguardano strettamente il funzionamento delle pagine, come per esempio l'id della risorsa da visualizzare, sia informazioni che possono modficare il comportamento, come per esempio il filtro preselezionato in una Search View.

A livello di server invece vengono utilizzate informazioni di sessione come _lang_ che influenza la configurazione della lingua selezionata. Inoltre ci è sempre possibile far riferimento al _context_ per aggiungere variabile legate alla nostra logica.

## Domain

I _domain_ in Odoo sono utilizzati per filtrare i dati. Utilizzano una sintassi specifica dell'ORM di Odoo e sostanzialmente vengono utilizzati per genereare la parte WHERE delle query SQL.

Un _domain_ è una lista di condizioni, ogni condizione è composta da una tupla

```python
(<nome_campo>, <operatore>, <valore>)
```

come per esempio:

```python
('is_done', '=', False)
```

Il _nome\_campo_ coniene una string che rappresenta il nome del campo, nei casi di campi relazionali si può fare ricorso alla dot-notation per accedere ai campi dell'elemento correlato.

Gli _opertori_ disponibili sono:

- **=**: uguale a
- **!=**: diverso da
- **>**: maggiore di 
- **>=**: maggiore o uguale a
- **<**: minore di 
- **<=**: minore o uguale a
- **=?**: ritorna True se il valore e' None o False, oppure si comporta come =
- **=like**: controlla il valore con un pattern. Un _ nel pattern sta per 'ogni carattere', una % sta per 'ogni stringa'
- **like**: simile a =like ma racchiude il valore in _%value%_
- **not like**: la negazione di like
- **ilike**: come like ma senza tenere conto di maiuscolo/minuscolo
- **not ilike**: la negazione di ilike
- **=ilike**: come ilike ma senza tenere conto di maiuscolo/minuscolo
- **in**: ritorna vero se il valore e' uguale a almneno uno dei valori contenuti nella lista di comparazione
- **not in**: ritorna vero se il valore e' diverso da tutti i valori contenuti nella lista di comparazione
- **child_of**: ritorna vero se e' un figlio dell'oggetto passato (controllonado il campo speciale _parent\_id_)

Nell'espressione dei domini possono essere utilizzati gli operatori logigici or, and e not rispettivamente con i simboli _|, &, !_. I domini fanno ricorso alla notazione prefissa per esprimere le connessioni logiche.

Per esempio per dire:

_Tutti i todo in stato done oppure tutti i todo assegnati all'utente 3 con titolo contenento 'Odoo'_

si esprime come

```python
[
    '|',('is_done','=',True),
        '&', ('user_id', '=', 3),
             ('name','like','Odoo')
]
```

che puo' essere rappresentato come

{{<mermaid align="left">}}
graph TD;
    C{OR}
    C --> D{AND}
    C --> E[Todo in stato done]
    D --> F[user_id = 3] 
    D --> G[name like Odoo] 
{{< /mermaid >}}


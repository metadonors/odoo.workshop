---
title: "Preambolo"
date: 2018-06-28T16:39:19+02:00
weight: 1
---

Per andare a esplorare cosa possiamo fare con i nostri modelli prepariamoci prima un nuovo modulo che estende gli esistenti. Per analizzare come funzionano le relazioni aggiungeremo anche un paio di modelli.

- **TodoProjects** progetti per raggruppare i task
- **TodoTag** tag per poter classificare i vari task

Abbiamo gia' un modulo preparato con queste modifiche, l'unica cosa che dobbiamo fare è scaricarlo nella nostra directory _addons_.

Per farlo, da terminale, entriamo nella cartella _addons_

```
    $ cd addons/
```

e diamo il comando _git_ per scaricare il modulo

```
    $ git submodule add https://github.com/metadonors/odoo.workshop.todo_plus.git todo_plus
```

A questo punto il modulo è disponibile per essere installato, andiamo nella sezione App di Odoo, aggiorniamo la lista e cerchiamo tra i moduli 'todo'

![todoplus](/odoo.workshop/screen/models_preambolo/todoplus.png?width=60pc)

Clicchiamo su _Install_ e navigiamo di nuovo nella nostra app Todo. Notiamo che il nuovo modulo ci ha aggiunto il sottomenu 'Progetti' dove e', appunto, possibile gestire i nostri progetti todo.

![progetti](/odoo.workshop/screen/models_preambolo/progetti.png?width=60pc)


### Continua

Adesso possiamo cominciare a vedere quali sono i [campi base](/odoo.workshop/models/campi_base/) dei modelli Odoo.
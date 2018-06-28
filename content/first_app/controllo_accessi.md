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



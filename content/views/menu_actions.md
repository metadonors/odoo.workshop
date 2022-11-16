---
title: "MenuItem e Actions"
date: 2018-07-02T16:17:22+02:00
weight: 1
---

## Gli oggetti del Menu

I _MenuItem_ sono oggetti salvati nella modello _ir.ui.menu_ and possono essere visualizzati nel menu tecnico:

**Settings -> Technical -> User Interface -> Menu Items**

Sono organizzati in una struttura ad albero. I menu radice (top-level) vengono listati nell'elenco completo delle applicazioni, ogni applicazione deve quindi avere almeno un elemento del menu per poter essere visualizzata.

Il primo sottolivello di menu crea il menu principale dell'applicazione, i successivi vengono rappresentati come menu dropdown nei rispettivi padri.

Un esempio di menu di secondo livello che abbiamo aggiunto con il nostro modulo _task\_plus_

```xml
<record model="ir.ui.menu" id="menu_todo_project">
    <field name="name">Projects</field>
    <field name="action" ref="action_todo_project" />
    <field name="parent_id" ref="todo_app.menu_todo_task" />
    <field name="sequence" eval="2" />
</record>
```

## Window Actions

Una _Window Action_ comanda le azioni che possono essere compiute attraverso l'interfaccia utente e solitamente vengono associate a dei _MenuItem_ oppure a dei bottoni. Principalmente comunicano alla UI su quale tipo di modello andare a lavorare e quali viste del modello rendere disponibili all'utente. Ad ogni azione può opzionalmente essere legato un _domain_ e un _context_ che andranno a modificare il comportamente specifico del sistema (vedi la [prossima sezione](/odoo.workshop/views/context_domain/))

Sempre nel nostro modulo _task\_plus_ abbiamo legato al menu dei progetti questa action:

```xml
<record model="ir.actions.act_window" id="action_todo_project">
    <field name="name">Projects</field>
    <field name="res_model">todo.project</field>
    <field name="view_mode">tree,form</field>
    <field name="domain">[]</field>
    <field name="context">{}</field>
</record>
```
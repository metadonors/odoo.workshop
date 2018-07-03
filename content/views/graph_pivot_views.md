---
title: "Pivot e Graph View"
date: 2018-07-02T16:18:06+02:00
weight: 5
---

## Pivot

I pivot sono un potente strumento di analisi dei dati fornito da Odoo. Una vista pivot rappresenta i dati in una matrice dinamica.

Un esempio di vista Pivot

```xml
<record id='view_pivot_todo_task' model='ir.ui.view'>
    <field name="name">todo.task.pivot</field>
    <field name="model">todo.task</field>
    <field name='arch' type='xml'>
        <pivot>
            <field name="user_id" type='col'>
            <field name="work_hours" type='measure'>
        </pivot>
    </field>
</record>
```

## Graph

Le Graph View rappresentano i dati in forma di grafici

Un esempio

```xml
<record id='view_graph_todo_task' model='ir.ui.view'>
    <field name="name">todo.task.graph</field>
    <field name="model">todo.task</field>
    <field name='arch' type='xml'>
        <graph>
            <field name="work_hours" type='measure'>
        </graph>
    </field>
</record>
```

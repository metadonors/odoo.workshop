---
title: "Panoramica"
date: 2018-07-02T12:16:01+02:00
weight: 1

---

Il server di Odoo fornisce un API esterna che può essere utilizzata da applicazioni terze per integrarsi. L'API sono le stesse che utilizza il client web ufficiale di Odoo per funzionare. Quindi qualsiasi funzionalità utlizzata nel client web può essere utilizzita direttamente tramite API.

Le API di Odoo supportano i protocolli XML-RPC e JSON-RPC, è quindi possibili scrivere integrazioni in qualsiasi linguaggio che supporti questi due protocolli. Per esempio nella [documentazione ufficiale](https://www.odoo.com/documentation/11.0/webservices/odoo.html) ci sono esempi per: Python, PHP, Ruby e Java.

Per questioni di chiarezza continueremo con i nostri esempi in Python. Inoltre faremo ricorso a una libreria python specifica che crea un'astrazione sopra le API base per ricreare un'interfaccia quanto più possibile a quella utilizzata direttamente all'interno del framework.

La libreria a cui faremo ricorso è [OdooRPC](https://pypi.org/project/OdooRPC/)

Solitamente potete installarla nel vostro ambiente python con il comando 

```bash
$ pip install OdooRPC
```

Per semplificare però gli esempi di questo corso, abbiamo aggiungo un comando specifico alla nostra immagine docker che ci permette di avviare una shell interattiva _ipython_ con la libreria già installata. Per farlo apriamo un altro terminale e digitiamo:

```bash
$ cd odoo.dockerenv/
$ docker-compose run odoo ipython
```

Ci comparirà quindi il seguente output:

```python

odoo.dockerenv (completed) $ docker-compose run odoo ipython

Starting odoodockerenv_postgres_1 ... done
Python 3.5.3 (default, Jan 19 2017, 14:11:04) 
Type 'copyright', 'credits' or 'license' for more information
IPython 6.4.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: import odoorpc

In [2]: odoorpc
Out[2]: <module 'odoorpc' from '/usr/local/lib/python3.5/dist-packages/odoorpc/__init__.py'>
```

### Continua

A questo punto siamo in grado di effettuare le nostre chiamate, vediamo nel dettaglio [come procedere](/odoo.workshop/api/operazioni/)
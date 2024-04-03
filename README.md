# k8s-provisioner
----

Questo progetto contiene una serie di ruoli ansible, che uniti ad un playbook di avvio e ad un inventory, settato adeguatamente, crea e configura un cluster k3s su vmware.

## Automazioni realizzate dal playbook

* Creazione delle vm tramite terraform, come indicate da inventory, su vmware
* Configurazione DNS per le macchine
* Configurazione firewall per le macchine
* Attivazione protect-kernel-defaults sulle macchine
* Installazione k3s sulle macchine, sia master che worker
* Download kubeconfig creato sulla directory del playbook
* Creazione namespace sul cluster con terraform
* Installazione tekton operator
* Installazione tekton dashboard e sua esposizione tramite ingress

## LoadBalancing

Viene utilizzato il loadbalancer di default di k3s

## Requirements

* Ambiente VMWare
* ansible
* terraform
* template da cui verranno create le vm deve contenere una chiave ssh per l'accesso diretto
* l'utente con cui si accede deve avere permessi di sudo:NOPASSWD

### Configurazione VMs tramite inventory

Viene fornito un esempio di inventory esplicativo per la configurazione delle vm (inventory.yaml)

## VARIABILI
Le seguenti variabile andranno modificare per adattare l'automazione a funzionare sul vostro ambiente.
Tutte le variabili indicate sono obbligatorie e possono essere passate al playbook tramite file

|Variable|Description|Default|
|:---|:---|:---|
|vsphere_user|Username per connettersi a vmware|NO|
|vsphere_password|Password per connetterci a vmware|NO|
|vsphere_server|hostname del vcenter|NO|
|vsphere_allow_unverified_ssl|Check del certificato del vcenter (true:false)|NO|
|vsphere_datacenter|vsphere datacenter|NO|
|vsphere_compute_cluster|vsphere computer cluster|NO|
|vsphere_datastore|vsphere datastore|NO|
|vsphere_network|vsphere network|NO|
|vsphere_virtual_template|Template da cui creare le vm|NO|
|vsphere_folder|Folder in cui verranno create le vm|NO|
|vms_domain|Dominio delle vm|NO|
|namespaces|Lista di namespace da creare al provision del cluster|NO|
|tekton_hostname|Vhost dell'ingress per la tekton dashboard|NO|

#### Esecuzione del playbook

Per eseguire il playbook lasciare questo comando:

```console
ansible-playbook fire.yaml -i inventory -e @vars.yaml
```
Dove vars.yaml è il file contente le vostre variabili.

Alla fine di esecuzione del playbook per eseguire kubectl eseguire questa export:

```console
export KUBECONFIG=/path/to/k3s.yaml
```

# k8s-provisioner
----

Questo progetto contiene una serie di ruoli ansible, che uniti ad un playbook di avvio e ad un inventory, settato adeguatamente, crea e configura un cluster k3s su vmware.
Il progetto contiene una parte helm che esegue il deployment di un'applicativo molto semplice composto da:
  - database mysql
  - api node
  - htlm + javascirpt frontend (nginx)

## Automazioni realizzate dal playbook

* Creazione delle vm tramite terraform, come indicate da inventory, su vmware
* Configurazione DNS per le macchine
* Configurazione firewall per le macchine
* Attivazione protect-kernel-defaults sulle macchine
* Installazione k3s sulle macchine, sia master che worker
* Download kubeconfig creato sulla directory del playbook
* Creazione namespace sul cluster con terraform
* Creazione job kube-bench sul namespace default (assesment di security)
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

----
## HELM

### Requirements
* L'utente deve fornire risoluzione DNS per gli ingress creati
* helm

### Installazione applicativo custom

Nella cartella helm esistono tutti i template per il deployment dell'applicativo sopra menzionato.
Esiste un file values.yaml riempito con delle variabili d'esmepio.

Per installare lìapplicativo eseguire:
```console
kubectl create namespace NAMESPACE_NAME
```

```console
cd helm/nginx-node-mysql && helm install RELEASE_NAME . -n NAMESPACE_NAME
```

Eseguendo 
```console
kubectl get pod -n NAMESPACE_NAME
```
Si ha uno stato dei pod deployati.

Dopo il deployment eseguire sul container mysql, una volta entranti con la cli mysql, questo comando per permettere l'autenticazione da parte del pod node:
Sostituire USERNAME e PASSWORD con quelle settate vie helm

```console
ALTER USER 'USERNAME' IDENTIFIED WITH mysql_native_password BY 'PASSWORD';
flush privileges;
```

### Installazione AWX Operator
doc: https://ansible.readthedocs.io/projects/awx-operator/en/latest/
github: https://github.com/ansible/awx-operator

Eseguire:

```console
helm repo add awx-operator https://ansible.github.io/awx-operator/
helm repo update
helm install -n awx --create-namespace my-awx-operator awx-operator/awx-operator
```
Verrà installato l'awx operator che rende disponibile una nuova CRD che installa a sua volta AWX.

Per installare AWX con la CRD è fornito un semplicissimo helm 

```console
cd helm/awx && helm install RELEASE_NAME . -n NAMESPACE_NAME
```
Nella cartella c'è un file values.yaml con le variabili d'esempio con cui creare lo stack awx.

## Pipeline CICD

All'interno della cartella yaml_manifests si pososno trovare delle pipeline di esempio per l'applicativo custom.
Le pipeline partendo da una git clone eseguono la build dei due container e poi il push verso un registry.

Per rendere le pipeline funzionanti, oltre a tekton installato (dal playbook) sono necessari tre componenti:
* secret per le auth del registry
* secret con chiave ssh e known_host del vostor repository
* 1 persistentVolumeClaim da 1 GB

Nella cartella yaml_manifest sono forniti due secret di esempio, vuoti, da poter utilizzare con le proprie credenziali.

----
## KUBE-BENCH

Kube-Bench è uno strumento open-source progettato per valutare la sicurezza di un cluster Kubernetes. Esso esegue controlli su vari aspetti di sicurezza secondo le linee guida di sicurezza Kubernetes, aiutando gli amministratori di sistema a identificare vulnerabilità e configurazioni non sicure nel loro ambiente Kubernetes.

Perché usare Kube-Bench?
1. Identificazione di vulnerabilità
Kube-Bench esegue controlli su numerosi aspetti critici della configurazione di Kubernetes, rilevando potenziali vulnerabilità di sicurezza e configurazioni non sicure.

2. Conformità alle best practice
Molti standard di sicurezza, come CIS Kubernetes Benchmark, offrono linee guida per configurare Kubernetes in modo sicuro. Kube-Bench consente di valutare rapidamente la conformità del cluster rispetto a tali best practice.

3. Automatizzazione
Kube-Bench automatizza il processo di valutazione della sicurezza del cluster, risparmiando tempo e sforzi agli amministratori di sistema. È possibile integrare Kube-Bench in pipeline di CI/CD o schedulare controlli periodici per garantire una sicurezza continua.

4. Reporting dettagliato
Kube-Bench fornisce report dettagliati che evidenziano le aree di miglioramento e le potenziali minacce. Questi report sono utili per pianificare e prioritizzare le azioni correttive.

5. Supporto per ambienti multi-cloud
Kube-Bench è progettato per essere utilizzato su diversi provider cloud e configurazioni di Kubernetes, consentendo agli utenti di valutare la sicurezza del cluster indipendentemente dall'ambiente di hosting.




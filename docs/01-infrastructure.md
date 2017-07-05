# VM provisioning

In diesem Laboraufbau werden die notwendigen Systeme in einem VSphere-Cluster erstellt. Es werden 6 Maschinen erzeugt. 

Nach Fertigstellung der beschriebenen Schritte sollten die folgenden VMs existieren:

```
hammer> host list --search kube
```

````
----|---------------------------------------------|------------------|---------------|--------------
ID  | NAME                                        | BETRIEBSSYSTEM   | HOSTGRUPPE    | IP           
----|---------------------------------------------|------------------|---------------|--------------
112 | oobtest-kubemaster-1-test-01.test.bnotk.net | Ubuntu 16.04 LTS | test/oob-test | 10.1.232.5   
113 | oobtest-kubemaster-2-test-01.test.bnotk.net | Ubuntu 16.04 LTS | test/oob-test | 10.1.232.6   
114 | oobtest-kubemaster-3-test-01.test.bnotk.net | Ubuntu 16.04 LTS | test/oob-test | 10.1.232.7   
116 | oobtest-kubenode-1-test-01.test.bnotk.net   | Ubuntu 16.04 LTS | test/oob-test | 10.1.232.8   
118 | oobtest-kubenode-2-test-01.test.bnotk.net   | Ubuntu 16.04 LTS | test/oob-test | 10.1.232.9   
117 | oobtest-kubenode-3-test-01.test.bnotk.net   | Ubuntu 16.04 LTS | test/oob-test | 10.1.232.10  
----|---------------------------------------------|------------------|---------------|--------------
````

> Alle Maschinen werden mit DHCP-Reservierungen erstellt. Das erleichtert den Bootstrap-Prozess erheblich.

Für die externe Erreichbarkeit wird später ein Load Balancer konfiguriert. Hierfür muss eine extra IP reserviert werden. Hinter diesem LB werden die 3 Kubernetes controller gestellt.

## Voraussetzungen

Die Maschinen werden über [foreman](https://foreman.bnotk.net) erstellt. Nachfolgend werden die notwendigen Konfigurationen der einzelnen Tab in "Create Host"-Dialog genannt:

* Host
  * Name: oobtest-kube{master,node}-{1..3}-test-01
  * Host Group: test/oob-test
  * Deploy on: vmware(VMware)
  * Environment: kubernetes
* Operation System
  * Architecture: x86_64
 * Operation system: Ubuntu 16.04 LTS
 * Provisioning Method: Image Based
* Virtual Machine
  * CPUs: 2
  * Memory (MB): 2048
  * Storage
    * Size (GB) 25

Alle nicht genannten Option verbleiben auf den angebotenen Defaults.

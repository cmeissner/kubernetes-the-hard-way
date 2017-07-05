# Kubernetes The Hard Way

Dieser Guide zeigt, wie man Kubernetes auf die harte Tour aufsetzt. Wer einen Cluster vollautomatisch aufsetzen möchte, der ist hier falsch. 

Mit diesem Guide lernt man viel über die Zusammenhänge und über den Weg, den man beschreitet, wenn man einen Kubernetes-Cluster bootsrapen muss oder möchte.

> Der erstellte Cluster ist nicht für den produktiven Einsatz vorgesehen. 

## Zielgruppe

Zur Zielgrupper dieses Guides gehören alle Interessierten, die erfahren wollen, wie die einzelnen Bestandteile in einander greifen. Wenn dieser Guide abgearbeitet wurde, sollte man in der Lage sein eine automatische Installation abzubilden.

## Cluster Details

* Kubernetes latest (1.7.0)
* Docker latest (1.29)
* etcd latest (3.1.4)
* [CNI basierte Netzwerkstruktur](https://github.com/containernetworking/cni)
* Secure communication between all components (etcd, control plane, workers)
* Default Service Account and Secrets
* [RBAC authorization](https://kubernetes.io/docs/admin/authorization)
* [TLS client certificate bootstrapping for kubelets](https://kubernetes.io/docs/admin/kubelet-tls-bootstrapping)
* Overlay-Netzwork (flannel)
* DNS add-on
* Kubernetes dashboard

### Was fehlt

Dem fertigen Cluster fehlen folgende Features:

* [Logging](https://kubernetes.io/docs/concepts/cluster-administration/logging/)
* [Cluster add-ons](https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)

## Laboraufbau

Dieses Tutorial basiert auf virtuelle Maschinen VMWare oder Virtualbox

* [VM Provisioning](docs/01-infrastructure.md)
* [Setting up a CA and TLS Cert Generation](docs/02-certificate-authority.md)
* [Setting up TLS Client Bootstrap and RBAC Authentication](docs/03-auth-configs.md)
* [Bootstrapping a H/A etcd cluster](docs/04-etcd.md)
* [Bootstrapping a H/A Kubernetes Control Plane](docs/05-kubernetes-controller.md)
* [Bootstrapping Kubernetes Workers](docs/06-kubernetes-worker.md)
* [Configuring the Kubernetes Client - Remote Access](docs/07-kubectl.md)
* [Managing the Container Network Routes](docs/08-network.md)
* [Deploying the Cluster DNS Add-on](docs/09-dns-addon.md)
* [Smoke Test](docs/10-smoke-test.md)
* [Cleaning Up](docs/11-cleanup.md)

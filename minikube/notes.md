# Minikube Notes

## Installation

Install Hyperkit driver:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/docker-machine-driver-hyperkit \
&& chmod +x docker-machine-driver-hyperkit \
&& sudo mv docker-machine-driver-hyperkit /usr/local/bin/ \
&& sudo chown root:wheel /usr/local/bin/docker-machine-driver-hyperkit \
&& sudo chmod u+s /usr/local/bin/docker-machine-driver-hyperki
```

Install Minikube from `brew`:

```bash
brew cask install minikube
```

Start Minikube:

```bash
minikube start --vm-driver hyperkit
```

Start the dashboard:

```bash
minikube dashboard
```

## Mess around with hello-minikube

```bash
$ kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port 8080
$ kubectl expose deployment hello-minikube --type=NodePort
$ kubectl get pod
$ curl $(minikube service hello-minikube --url)
$ kubectl delete service hello-minikube
$ kubectl delete deployment hello-minikube
```

## Persist storage

Dashboard > Create

Paste the following YAML:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0001
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 5Gi
  hostPath:
    path: /Users/youweam/minikube/data/pv0001/
```

Click "Upload".

## Create a custom namespace

Make trashing things a little easier when dev inevitably gets out of control and you can't roll back your own mess:

```bash
$ kubectl create namespace <name>
```

## Start Minikube with a local registry and the Hyperkit driver

```bash
$ minikube start --vm-driver hyperkit --insecure-registry localhost:5000
$ eval $(minikube docker-env)
```

Create a local registry container with Docker (boots into minikube VM because of the `eval` command):

```bash
$ docker run -d -p 5000:5000 --restart=always --name registry   -v /data/docker-registry:/var/lib/registry registry:2
```

## Add a rethink image to the local registry

```bash
$ docker pull rethinkdb
$ docker tag rethinkdb localhost:5000/xnode/rethinkdb-2.3.6
$ docker push localhost:5000/xnode/rethinkdb-2.3.6
```

RethinkDB can be booted into minikube with:

```yaml
spec:
  containers:
  - image: localhost:5000/xnode/rethinkdb-2.3.6
    name: rethinkdb
```

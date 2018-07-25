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

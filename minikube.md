#### Install minikube on macOS
https://minikube.sigs.k8s.io/docs/start/

```
brew install hyperkit
```

```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64
sudo install minikube-darwin-amd64 /usr/local/bin/minikube
```

```
ln -fs $(which minikube) /usr/local/bin/kubectl
```

```
minikube config set driver hyperkit
minikube config set cpus 4
minikube config set memory 16GB
minikube config set disk-size 120GB
```

```
minikube config get driver
minikube config get cpus
minikube config get memory
minikube config get disk-size
```

```
minikube start --kubernetes-version=$(curl -L -s https://dl.k8s.io/release/stable.txt)
```

```
minikube addons enable ingress
```

```
minikube ssh
```

```
minikube dashboard
```

```
sudo minikube tunnel
```

```
ssh -J docker@$(minikube ip) <USER>@<POD_IP>
```

#### minikube
```
brew install minikube
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
minikube addons enable ingress
```

```
minikube start
```

```
minikube ssh
```

```
minikube dashboard
```

```
minikube tunnel
```

```
ssh -J docker@$(minikube ip) <USER>@<POD_IP>
```

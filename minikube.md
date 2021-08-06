#### minikube
```
brew install minikube
```

```
minikube config set driver hyperkit
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

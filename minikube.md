#### minikube
```
brew install minikube
```

```
minikube config set driver hyperkit
```

```
minikube ssh
```

```
minikube start
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

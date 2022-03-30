```sudo apt install -yq golang-cfssl```

```shell
cat <<EOF | cfssl genkey - | cfssljson -bare server
{
  "hosts": [
    "kubernetes.docker.internal"
  ],
  "CN": "kubernetes.docker.internal",
  "key": {
    "algo": "ecdsa",
    "size": 256
  }
}
EOF
```

```shell
cat <<EOF | cfssl gencert -initca - | cfssljson -bare ca
{
  "CN": "k8s.io",
  "key": {
    "algo": "rsa",
    "size": 2048
  }
}
EOF
```

```shell
kubectl apply -f - <<EOL
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: app-csr
  namespace: k8s
spec:
  request: $(cat server.csr | base64 | tr -d '\n')
  signerName: kubernetes.docker.internal/serving
  usages:
  - digital signature
  - key encipherment
  - server auth
EOL
```

```
kubectl -n k8s certificate approve app-csr
```

```shell
kubectl -n k8s get csr app-csr -o jsonpath='{.spec.request}' | \
  base64 --decode | \
  cfssl sign -ca ca.pem -ca-key ca-key.pem -config server-signing-config.json - | \
  cfssljson -bare ca-signed-server
```

```shell
kubectl -n k8s get csr app-csr -o json | \
  jq '.status.certificate = "'$(base64 ca-signed-server.pem | tr -d '\n')'"' | \
  kubectl replace --raw /apis/certificates.k8s.io/v1/certificatesigningrequests/app-csr/status -f -
```

```shell
kubectl -n k8s get csr app-csr -o jsonpath='{.status.certificate}' \
    | base64 --decode > server.crt
````

```shell
kubectl -n k8s create secret tls server --cert server.crt --key server-key.pem
````

```shell
kubectl -n k8s create configmap serving-ca --from-file ca.crt=ca.pem
```
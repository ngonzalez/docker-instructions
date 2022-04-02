#### Install cfssl

```shell
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.1/cfssl_1.6.1_linux_amd64 -O cfssl
chmod +x cfssl
sudo mv cfssl /usr/local/bin
ln -fs /usr/local/bin/cfssl /usr/bin/cfssl
```

```shell
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.1/cfssljson_1.6.1_linux_amd64 -O cfssljson
chmod +x cfssljson
sudo mv cfssljson /usr/local/bin
ln -fs /usr/local/bin/cfssljson /usr/bin/cfssljson
```

#### Generate certificates

```shell
cat <<EOF | cfssl genkey - | cfssljson -bare server
{
  "hosts": [
    "kubernetes.docker.internal"
  ],
  "CN": "nginx-server",
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
  "CN": "nginx-server",
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
  name: nginx-server
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

```shell
cat > server-signing-config.json <<EOL
{
    "signing": {
        "default": {
            "usages": [
                "digital signature",
                "key encipherment",
                "server auth"
            ],
            "expiry": "876000h",
            "ca_constraint": {
                "is_ca": false
            }
        }
    }
}
EOL
```

```
kubectl -n k8s certificate approve nginx-server

kubectl -n k8s get csr nginx-server -o jsonpath='{.spec.request}' | \
  base64 --decode | \
  cfssl sign -ca ca.pem -ca-key ca-key.pem -config server-signing-config.json - | \
  cfssljson -bare ca-signed-server

kubectl -n k8s get csr nginx-server -o json | \
  jq '.status.certificate = "'$(base64 ca-signed-server.pem | tr -d '\n')'"' | \
  kubectl replace --raw /apis/certificates.k8s.io/v1/certificatesigningrequests/nginx-server/status -f -

kubectl -n k8s get csr nginx-server -o jsonpath='{.status.certificate}' \
    | base64 --decode > server.crt

kubectl -n k8s create secret tls server --cert server.crt --key server-key.pem

kubectl -n k8s create configmap serving-ca --from-file ca.crt=ca.pem
```

#### Verify certificate with openssl

```
openssl s_client -CAfile ca.pem -connect kubernetes.docker.internal:443
```

#### Create a chain certificate for Nginx
https://nginx.org/en/docs/http/configuring_https_servers.html

```shell
cat server.crt ca-signed-server.csr ca-signed-server.pem > server.chained.crt
```

#### Configure Nginx

```
ssl_certificate /home/debian/server.chained.crt;
ssl_certificate_key /home/debian/server-key.pem;
```

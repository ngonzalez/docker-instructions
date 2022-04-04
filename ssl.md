#### Install cfssl
```shell
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.1/cfssl_1.6.1_linux_amd64 -O cfssl
chmod +x cfssl
sudo mv cfssl /usr/local/bin
sudo ln -fs /usr/local/bin/cfssl /usr/bin/cfssl
```

```shell
wget https://github.com/cloudflare/cfssl/releases/download/v1.6.1/cfssljson_1.6.1_linux_amd64 -O cfssljson
chmod +x cfssljson
sudo mv cfssljson /usr/local/bin
sudo ln -fs /usr/local/bin/cfssljson /usr/bin/cfssljson
```

#### Create root ca
```shell
cat <<EOF | cfssl gencert -initca - | cfssljson -bare ca
{
  "CN": "link12.ddns.net",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [{
    "C": "FR",
    "L": "Paris",
    "O": "Ngonzalez",
    "OU": "Link12",
    "ST": "France"
  }],
  "ca": {
    "expiry": "42720h"
  }
}
EOF
```

#### Create server certificate
```shell
cat <<EOF | cfssl genkey - | cfssljson -bare server
{
  "CN": "link12.ddns.net",
  "key": {
    "algo": "ecdsa",
    "size": 256
  },
  "names": [{
    "C": "FR",
    "L": "Paris",
    "O": "Ngonzalez",
    "OU": "Link12",
    "ST": "France"
  }],
  "hosts": [
    "link12.ddns.net"
  ]
}
EOF
```

#### Sign server certificate
```shell
cat > server-signing-config.json <<EOL
{
    "signing": {
        "default": {
            "expiry": "8760h"
        },
        "profiles": {
            "server": {
                "usages": [
                    "digital signature",
                    "key encipherment",
                    "server auth"
                ],
                "expiry": "70080h",
                "ca_constraint": {
                    "is_ca": false
                }
            }
        }
    }
}
EOL
```

```
kubectl -n k8s delete csr link12.ddns.net
```

```
kubectl apply -f - <<EOL
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  namespace: k8s
  name: link12.ddns.net
spec:
  request: $(cat server.csr | base64 | tr -d '\n')
  signerName: link12.ddns.net/serving
  usages:
  - digital signature
  - key encipherment
  - server auth
EOL
```

```
kubectl -n k8s certificate approve link12.ddns.net
```

```
kubectl -n k8s get csr link12.ddns.net -o jsonpath='{.spec.request}' | \
  base64 --decode | \
  cfssl sign -ca ca.pem -ca-key ca-key.pem -config server-signing-config.json -profile server - | \
  cfssljson -bare ca-signed-server

kubectl -n k8s get csr link12.ddns.net -o json | \
  jq '.status.certificate = "'$(base64 ca-signed-server.pem | tr -d '\n')'"' | \
  kubectl replace --raw /apis/certificates.k8s.io/v1/certificatesigningrequests/link12.ddns.net/status -f -

kubectl -n k8s get csr link12.ddns.net -o jsonpath='{.status.certificate}' \
  | base64 --decode > server.crt
````

#### Create secret
```shell
kubectl -n k8s delete secret server
```

```shell
kubectl -n k8s create secret tls server --cert server.crt --key server-key.pem
```

```shell
kubectl -n k8s get secret server
```

#### Create configmap
```shell
kubectl -n k8s delete configmap serving-ca
```

```shell
kubectl -n k8s create configmap serving-ca --from-file ca.crt=ca.pem
```

```shell
kubectl -n k8s get configmap serving-ca
```

#### bundle with cfssl
```shell
cfssl bundle -domain link12.ddns.net \
             -cert server.crt \
             -key server-key.pem \
             -ca-bundle ca.pem \
             > bundle.json

cat bundle.json | jq .bundle -r > bundle.crt
cat bundle.json | jq .key -r > bundle-key.pem
```

#### Configure Nginx

```
ssl_certificate /etc/ssl/certs/bundle.crt
ssl_certificate_key /etc/ssl/private/bundle-key.pem
```

#### Try to connect with openssl

```shell
openssl s_client -CAfile ca.pem -connect link12.ddns.net:443
```

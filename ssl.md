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

#### Create intermediate ca
```shell
cat <<EOF | cfssl gencert -initca - | cfssljson -bare intermediate_ca
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

#### Sign intermediate ca
```shell
cat > intermediate-signing-config.json <<EOL
{
    "signing": {
        "default": {
            "expiry": "8760h"
        },
        "profiles": {
            "intermediate": {
                "usages": [
                    "cert sign", 
                    "crl sign"
                ],
                "expiry": "70080h",
                "ca_constraint": {
                    "is_ca": true,
                    "max_path_len": 1
                }
            }
        }
    }
}
EOL
```

```shell
cat intermediate_ca.csr | cfssl sign -ca ca.pem -ca-key ca-key.pem -config intermediate-signing-config.json -profile intermediate - | \
cfssljson -bare intermediate
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

```shell
cat server.csr | cfssl sign -ca ca.pem -ca-key ca-key.pem -config server-signing-config.json -profile server - | \
cfssljson -bare server
```

#### bundle with cfssl
```shell
cfssl bundle -domain link12.ddns.net \
             -cert server.pem \
             -key server-key.pem \
             -ca-bundle ca.pem \
             -int-bundle intermediate.pem \
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

1. Generate the certificates

# Generate CA key and certificate
```bash
openssl genrsa -out ca-key.pem 2048
openssl req -x509 -new -nodes -key ca-key.pem -sha256 -days 365 -out ca-cert.pem -subj "/CN=Test-CA"
```

# Generate server key and certificate
```bash
openssl genrsa -out server-key.pem 2048
openssl req -new -key server-key.pem -out server.csr -subj "/CN=backend"
openssl x509 -req -in server.csr -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -days 365 -sha256
```

# Generate Envoy key and certificate
```bash
cat > envoy-csr.cnf <<EOF
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
CN = localhost

[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost
EOF
```

```bash
openssl genrsa -out envoy-key.pem 2048
openssl req -new -key envoy-key.pem -out envoy.csr -config envoy-csr.cnf
openssl x509 -req -in envoy.csr -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -out envoy-cert.pem -days 365 -extensions v3_req -extfile envoy-csr.cnf
```

# Generate client key and certificate
```bash
openssl genrsa -out client-key.pem 2048
openssl req -new -key client-key.pem -out client.csr -subj "/CN=client"
openssl x509 -req -in client.csr -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -out client-cert.pem -days 365 -sha256
```

# Cleanup
```bash
rm *.csr *.srl
cd ..
````

# Commands
```
curl -v --cert ./certs/client-cert.pem --key ./certs/client-key.pem --cacert ./certs/ca-cert.pem https://envoy:8443

curl -v --cert ./certs/client-cert.pem --key ./certs/client-key.pem --cacert ./certs/ca-cert.pem https://localhost:8443

curl -vk --cert ./certs/client-cert.pem --key ./certs/client-key.pem  https://localhost:8443  (take note that we're using -k flag here since we don't pass in the ca cert)
```

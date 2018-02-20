# Client-side SSL with self signed certs demo

### Generate a CA
```bash
openssl genrsa -out ca.key 2048
```

### Generate a cert that'll expire in 10000 days
```bash
openssl req -x509 -new -nodes -key ca.key -days 10000 -out ca.crt -subj "/CN=example-ca"
```

### Generate a client key
```bash
openssl genrsa -out client.key 2048
```

### Generate a client certificate request (CSR)
```bash
openssl req -new -key client.key -out client.csr -subj "/CN=client1" -config openssl.cnf
```

### Generate a signed client certificate
```bash
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days 365 -extensions v3_req -extfile openssl.cnf
rm client1.csr 
```

### Test the client cert will work
```bash
openssl verify -verbose -CAfile ca.crt client.crt
```

### Optionally generate a `.pem` / `.p12` bundle for use in some clients
```bash
openssl pkcs12 -export -clcerts -in client.crt -inkey client.key -out client.p12 -passout pass:foo 
openssl pkcs12 -in client.p12 -out client.pem -clcerts -passin pass:foo -passout pass:foo 
```

### Deploy it all!
(depends on [kd](https://github.com/ukhomeoffice/kd))
```bash
export KUBE_NAMESPACE=my-namespace
export CA_CRT=$(base64 ca.crt)
export URL=myurl.tld.uk
export NAME=mydemo
kd -f secret.yml \
   -f deployment.yml \
   -f network-policy.yml \
   -f service.yml \
   -f ingress.yml
```

### Demo it works
```bash
curl -v --cert client.crt --key client.key https://${URL}
```
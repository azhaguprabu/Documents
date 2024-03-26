### Genreate the root CA key pair
```
mkdir /root/ca

cd /root/ca
mkdir certs crl newcerts private
chmod 700 private
touch index.txt
echo 1000 > serial

openssl genrsa -aes256 -out private/ca.key.pem 4096
chmod 400 private/ca.key.pem

openssl req -config openssl.cnf \
      -key private/ca.key.pem \
      -new -x509 -days 7300 -sha256 -extensions v3_ca \
      -out certs/ca.cert.pem

chmod 444 certs/ca.cert.pem

openssl x509 -noout -text -in certs/ca.cert.pem
```

### Generate Intermediate certificate
```
mkdir /root/ca/intermediate

cd /root/ca/intermediate
mkdir certs crl csr newcerts private
chmod 700 private
touch index.txt
echo 1000 > serial

echo 1000 > /root/ca/intermediate/crlnumber

cd /root/ca

openssl genrsa -aes256 \
      -out intermediate/private/intermediate.key.pem 4096
chmod 400 intermediate/private/intermediate.key.pem

openssl req -config intermediate/openssl.cnf -new -sha256 \
      -key intermediate/private/intermediate.key.pem \
      -out intermediate/csr/intermediate.csr.pem

openssl ca -config openssl.cnf -extensions v3_intermediate_ca \
      -days 3650 -notext -md sha256 \
      -in intermediate/csr/intermediate.csr.pem \
      -out intermediate/certs/intermediate.cert.pem

```

### Verify the intermediate certificate
```
openssl x509 -noout -text \
      -in intermediate/certs/intermediate.cert.pem

openssl verify -CAfile certs/ca.cert.pem \
      intermediate/certs/intermediate.cert.pem
```

### Server certificate Generate
```
openssl genrsa -out intermediate/private/server.pem 2048

chmod 400 intermediate/private/server.pem 
```

### CSR generate 
```
openssl req -config intermediate/openssl.cnf \
    -key intermediate/private/server.key \
    -new -out intermediate/csr/server.csr
```

### Sign Certificate
```
openssl ca -config intermediate/openssl.cnf \
    -extensions server_cert -days 375 -notext \
    -in intermediate/csr/server.csr \
    -out intermediate/certs/server.crt 

chmod 444 intermediate/certs/server.crt
```

### Verify Server Certificate
```
openssl x509 -noout -text \
    -in intermediate/certs/server.crt 

openssl verify -CAfile intermediate/certs/ca-chain.cert.pem \
    intermediate/certs/server.crt

```
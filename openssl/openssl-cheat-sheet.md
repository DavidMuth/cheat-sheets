# openssl SSL Certificates Cheat-Sheet

## Self-Signed Certificates

### Create CA

1. Generate RSA
```
openssl genrsa -aes256 -out ca-key.pem 4096
```
2. Generate the public Certificate for the CA
```
openssl req -new -x509 -sha256 -days 356 -key ca-key.pem -out ca.pem
```
### Generate Certificate
1. Create RSA key
```
openssl genrsa -out cert-key.pem 4096
```
2. Create a CSR (Certificate Signing Request)
```
openssl req -new -sha256 -subj "/CN=customcn" -key cert-key.pem -out cert.csr
```
3. Create extfile with alternative names (DNS, hostname, IP). For a wildcard Cert, only give the Domainname + TLD. 
```
echo "subjectAltName=DNS:custom-dns.record,IP:123.123.123.123" >> extfile.cnf
```
4. Create the Certificate
```
openssl x509 -req -sha256 -days 365 -in cert.csr -CA ca.pem -CAkey ca-key.pem -out cert.pem -extfile extfile.cnf -CAcreateserial
```
5. Combine the cert.pem and ca.pem files in a chain of trust
```
cat cert.pem > fullchain.pem
cat ca.pem >> fullchain.pem
```

## Convert Certificates

1. PEM to DER
```
openssl x509 -outform der -in cert.pem -out cert.der
```
2. DER to PEM
```
openssl x509 -inform der -in cert.der -out cert.pem
```
3. PFX to PEM
```
openssl pkcs12 -in cert.pfx -out cert.pem -nodes
```

## Install the CA as a trusted root CA
### Debian / Ubuntu
- Copy / move the CA certificate (`ca.crt`) to `/usr/local/share/ca-certificates/ca.crt`.
- Update the Cert Store with `sudo update-ca-certificates`

### Windows
-open Powershell as admin
```
Import-Certificate -FilePath "Path to ca.pem file" -CertStoreLocation Cert:\LocalMachine\Root
```

### Issues with Firefox
If the CA has been imported successfully, but Firefox still wont trust your Certificates do the following:
1. in Firefox open `about:config`
2. search for `security.enterprise_roots.enabled` and change the value to `true`.

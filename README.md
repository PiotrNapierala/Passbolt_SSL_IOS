
# Passbolt SSL Certificate on IOS 

IOS has strict rules regarding SSL certificates. Follow the steps below to install an IOS-compatible certificate for Passbolt.


## Requirements

- OpenSSL
## Openssl config file

```
nano openssl.cnf
```

Enter the data as shown below, replacing the values with the correct ones depending on your region.

```
[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
x509_extensions    = v3_req
prompt             = no

[ req_distinguished_name ]
C  = US
ST = State
L  = City
O  = Organization
OU = Organizational Unit
CN = Your_local_IP_address

[ v3_req ]
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
basicConstraints = CA:TRUE

[ alt_names ]
IP.1 = Your_local_IP_address
```


## Generate a private key and certificate

```
openssl genpkey -algorithm RSA -out server-key.pem -pkeyopt rsa_keygen_bits:2048
openssl req -x509 -new -nodes -key server-key.pem -sha256 -days 365 -out server-cert.pem -config openssl.cnf
```


## Generating rootCA

```
openssl genpkey -algorithm RSA -out rootCA-key.pem -pkeyopt rsa_keygen_bits:2048
openssl req -x509 -new -key rootCA-key.pem -sha256 -days 3650 -out rootCA-cert.pem -subj "/C=US/ST=State/L=City/O=Organization/OU=RootCA/CN=Root CA"
```

Remember to set the values according to your region.
## Signing the certificate

```
openssl req -new -key server-key.pem -out server.csr -config openssl.cnf
openssl x509 -req -in server.csr -CA rootCA-cert.pem -CAkey rootCA-key.pem -CAcreateserial -out server-cert.pem -days 365 -sha256 -extensions v3_req -extfile openssl.cnf
```
## IOS Configuration

After completing the above steps, download the files `server-cert.pem` and `rootCA-cert.pem` to your IOS device.

First, install `rootCA-cert.pem`, and then `server-cert.pem`.

Go to **Settings** -> **General** -> **About** -> **Certificates**, and trust the CA certificate.
## Configuring a new SSL certificate in Passbolt

Use the command below to apply changes to the Passbolt configuration.

```
sudo dpkg-reconfigure passbolt-ce-server
```

Skip creating a new database, then choose manual certificate installation in Nginx and provide the paths to `server-cert.pem` and `server-key.pem` files.
## Summary

After completing the above steps, your IOS device should accept the SSL certificate generated on the server. Once the certificate is trusted, the Passbolt application on IOS should function normally.

Tested on IOS 18.0.

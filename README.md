# b3c-load-balancer

Just a repo to store the set of scripts and configuration to start b3c's reverse proxy and load balancers.
This module is based on HAproxy (High Availability proxy) a fast and reliable load balancing reverse proxy.

# Installing HAProxy on Ubuntu 20.04

```
$ sudo apt install haproxy
$ haproxy -v
```

# Generating CA cert for local development

```
$ mkdir cert
$ cd cert
$ openssl genrsa -out jaristra.key 2048
$ openssl req -new -key jaristra.key -out jaristra.csr
$ openssl x509 -req -days 365 -in jaristra.csr -signkey jaristra.key -out jaristra.crt
$ sudo bash -c 'cat jaristra.key jaristra.crt >> /etc/ssl/private/jaristra.pem'
```

Resolving warning 'unable to load default 1024 bits DH parameter for certificate'

```
$ sudo openssl dhparam -out /etc/haproxy/dhparams.pem 2048

## Add these 2 lines to global in haproxy.cfg
ssl-default-server-ciphers PROFILE=SYSTEM
ssl-dh-param-file /etc/haproxy/dhparams.pem
```

Resolving the error 'cannot bind UNIX socket'

```
[ALERT] 032/043815 (2781) : Starting frontend GLOBAL: cannot bind UNIX socket [/run/haproxy/admin.sock]

Haproxy needs to write to /run/haproxy/admin.sock but it wont create the directory for you. Create the directory /run/haproxy/ first

$ sudo mkdir /run/haproxy
```

Cert details

```
country ..... : US
state ....... : New York
location .... : New York
company ..... : Jaristra LLC
org unit .... : b3c
fqdn ........ : b3c.jaristra.com.br
email ....... : b3c@jaristra.com.br
```

# Generating https cert

```
$ mkdir local
$ cd local
$ touch local.ext
```

local.ext content:

```
authorityKeyIdentifier = keyid,issuer
basicConstraints = CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = localhost
IP.1 = 127.0.0.1
```

Generate a second cert.key file

```
$ openssl genrsa -out localhost.key -des3 2048
```

Generate the certificate sign-in request (CSR) file

```
$ openssl req -new -key localhost.key -out localhost.csr
```

Sign-in this second cert using the root CA cert

```
$ openssl x509 -req -in localhost.csr \
  -CA ../CA.pem \
  -CAkey ../CA.key \
  -CAcreateserial -days 3650 -sha256 \
  -extfile localhost.ext \
  -out localhost.crt
```

Decrypt the csr key file to use on https servers

```
$ openssl rsa -in localhost.key -out localhost.decrypted.key
```

# References

```

[1] How to Get SSL HTTPS for Localhost
    https://www.section.io/engineering-education/how-to-get-ssl-https-for-localhost/

```

# Steps for Solution

## IP Addressing

**CA**
```
ip addr add 10.10.1.10/24 dev eth0
```

**Server**
```
ip addr add 10.10.1.20/24 dev eth0
```

**Client**
```
ip addr add 10.10.1.30/24 dev eth0
```

## CA: Start SSH

**CA**
```
start-ssh.sh
```

## CA: Root CA

**CA**
```
mkdir -p /root/ca/certs /root/ca/crl /root/ca/newcerts /root/ca/private
touch /root/ca/index.txt
echo 1000 > /root/ca/serial
```
```
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -out /root/ca/private/root-ca.key
```
```
openssl req -x509 -new -key /root/ca/private/root-ca.key -sha256 -days 3650 -config /etc/ssl/openssl.cnf -extensions v3_ca -out /root/ca/certs/root-ca.crt -subj "/C=AU/ST=QLD/O=CQUni/CN=Root CA"
```

## CA: Intermediate CA

**CA**
```
mkdir -p /root/ca/intermediate/certs /root/ca/intermediate/crl /root/ca/intermediate/csr /root/ca/intermediate/newcerts /root/ca/intermediate/private
```
```
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -out /root/ca/intermediate/private/intermediate.key
```
```
openssl req -new -key /root/ca/intermediate/private/intermediate.key -out /root/ca/intermediate/csr/intermediate.csr -subj "/C=AU/ST=QLD/O=CQUni/CN=Intermediate CA"
```
```
openssl x509 -req -in /root/ca/intermediate/csr/intermediate.csr -CA /root/ca/certs/root-ca.crt -CAkey /root/ca/private/root-ca.key -CAcreateserial -out /root/ca/intermediate/certs/intermediate.crt -days 1825 -sha256 -extensions v3_ca -extfile /etc/ssl/openssl.cnf
```

## Server: Start SSH, key, CSR

**Server**
```
start-ssh.sh
```
```
mkdir -p /etc/ssl/private /etc/ssl/certs
```
```
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out /etc/ssl/private/server.key
```
```
openssl req -new -key /etc/ssl/private/server.key -out /tmp/server.csr -subj "/C=AU/ST=QLD/O=CQUni/CN=www.12312491.lab"
```
```
printf 'subjectAltName=DNS:www.12312491.lab\nbasicConstraints=critical,CA:false\nkeyUsage=critical,digitalSignature,keyEncipherment\nextendedKeyUsage=serverAuth\n' > /tmp/server-ext.cnf
```

## Server: Copy CSR and extension file to CA

**Server**
```
scp /tmp/server.csr root@10.10.1.10:/tmp/server.csr
```
```
scp /tmp/server-ext.cnf root@10.10.1.10:/tmp/server-ext.cnf
```

## CA: Sign server certificate

**CA**
```
openssl x509 -req -in /tmp/server.csr -CA /root/ca/intermediate/certs/intermediate.crt -CAkey /root/ca/intermediate/private/intermediate.key -CAcreateserial -days 365 -sha256 -extfile /tmp/server-ext.cnf -out /tmp/server.crt
```
```
openssl x509 -in /tmp/server.crt -noout -text | grep -A2 "Subject Alternative Name"
```
```
cat /root/ca/intermediate/certs/intermediate.crt /root/ca/certs/root-ca.crt > /tmp/ca-chain.crt
```

## CA: Push server certificate and chain to Server

**CA**
```
scp /tmp/server.crt root@10.10.1.20:/tmp/server.crt
```
```
scp /tmp/ca-chain.crt root@10.10.1.20:/tmp/ca-chain.crt
```

## Server: Build fullchain, configure Nginx

**Server**
```
cat /tmp/server.crt /tmp/ca-chain.crt > /etc/ssl/certs/server-fullchain.crt
```
```
nano /etc/nginx/http.d/ssl.conf
```
```
server {
    listen 443 ssl;
    server_name www.12312491.lab;
    ssl_certificate /etc/ssl/certs/server-fullchain.crt;
    ssl_certificate_key /etc/ssl/private/server.key;
    root /var/www/html;
}
```
```
mkdir -p /var/www/html
nano /var/www/html/index.html
```
```
<h1>Week2 tutorial completed</h1>
```
```
nginx -t && (nginx -s reload || nginx)
```

## Client: Hosts entry and pull certificates

**Client**
```
echo "10.10.1.20 www.12312491.lab" >> /etc/hosts
```
```
scp root@10.10.1.10:/root/ca/certs/root-ca.crt /tmp/root-ca.crt
```
```
scp root@10.10.1.10:/root/ca/intermediate/certs/intermediate.crt /tmp/intermediate.crt
```
```
scp root@10.10.1.20:/tmp/server.crt /tmp/server.crt
```

## Client: Verify and test

**Client**
```
openssl x509 -in /tmp/server.crt -noout -text | grep -A2 "Subject Alternative Name"
```
```
openssl verify -CAfile /tmp/root-ca.crt -untrusted /tmp/intermediate.crt /tmp/server.crt
```
```
curl --cacert /tmp/root-ca.crt https://www.12312491.lab/
```
```
openssl s_client -connect www.12312491.lab:443 -CAfile /tmp/root-ca.crt
```

## Packet Capture

GNS3 topology view: right-click link between Switch1 and Server → Start capture

**Client**
```
curl --cacert /tmp/root-ca.crt https://www.12312491.lab/
```

GNS3 topology view: right-click link → Stop capture → save as `OpenSSL-CA-12312491-tls.pcap`

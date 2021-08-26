TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
(IANA)

```
sudo cat /etc/letsencrypt/live/brad.fi/README
This directory contains your keys and certificates.

`privkey.pem`  : the private key for your certificate.
`fullchain.pem`: the certificate file used in most server software.
`chain.pem`    : used for OCSP stapling in Nginx >=1.3.7.
`cert.pem`     : will break many server configurations, and should not be used
                 without reading further documentation (see link below).
```

Certbot
--must-staple
--staple-ocsp

Google 8.8.8.8
Quad9 9.9.9.9
OpenDNS 208.67.222.222
Cloudflare 1.1.1.1

ECDHE-RSA-AES128-SHA
/etc/letsencrypt/options-ssl-nginx.conf

https://ciphersuite.info/cs/TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA/

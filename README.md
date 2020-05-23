# Apache-http-reverse-proxy

### How to configure Apache 2 httpd as Reverse proxy in front of the tomcat java web application with SSL by Let's Encrypt?

```
Let’s Encrypt is a free, automated, and open certificate authority (CA), run for the public’s benefit. It is a service provided by the Internet Security Research Group (ISRG).

We give people the digital certificates they need in order to enable HTTPS (SSL/TLS) for websites, for free, in the most user-friendly way we can. We do this because we want to create a more secure and privacy-respecting Web.
```
`for more info`: https://letsencrypt.org/

![alt text](https://letsencrypt.org/images/le-logo-twitter.png)

You will cofigure the firewalld to allow only HTTPS port 443 and then the HTTPd will handle the request and redirect it to the tomcat internal server which is working on port 8080

1. Install and configure Let's encrypt with apache-httpd

```
# yum install certbot python2-certbot-apache
```
, The installed **SSL Certificate** file in path `/etc/letsencrypt/live/exmple.com/fullchain.pem`

, The installed **SSLCertificateKeyFile** `/etc/letsencrypt/live/exmple.com/privkey.pem`

2. Configure the `firewalld` to allow only HTTPS port 443

```
# firewall-cmd --permanent --zone=public --remove-port=80/tcp
# firewall-cmd --permanent --zone=public --remove-port=8080/tcp 
# firewall-cmd --permanent --zone=public --add-port=443/tcp
# firewall-cmd --permanent --zone=public --remove-service=http
# firewall-cmd --permanent --zone=public --add-service=https
# firewall-cmd --reload 
```

3. Configure apache-httpd will handle the request and redirect it to the tomcat internal server which is working on port 8080 in `/etc/httpd/conf.d/example.conf`

This will redirect the incoming request on port 443 to tomcat which is listening on port 8080

```
        <VirtualHost *:443>
          ServerName exmple.com
          ServerAlias *exmple.com
          ServerAdmin user@gmail.com
          ProxyPreserveHost On
          ProxyRequests off
          AllowEncodedSlashes NoDecode

          <Proxy *>
             Order deny,allow
             Allow from all 
          </Proxy>

          SSLEngine on
          SSLCertificateFile /etc/letsencrypt/live/exmple.com/fullchain.pem
          SSLCertificateKeyFile /etc/letsencrypt/live/exmple.com/privkey.pem
          Include /etc/letsencrypt/options-ssl-apache.conf

          ProxyPass / http://localhost:8080/ nocanon
          ProxyPassReverse / http://localhost:8080/

          LogLevel debug
          ErrorLog ${APACHE_LOG_DIR}/error.log
          CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>
```

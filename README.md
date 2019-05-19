# http-reverse-proxy

The configure file for apply Apache2 httpd as Reverse proxy in front of the tomcat java web application with https support using SSL provided by letsencrypt for more info: https://letsencrypt.org/

![alt text](https://letsencrypt.org/images/le-logo-twitter.png)

You will cofigure the firewalld to allow only HTTPS port 443 and then the HTTPd will handle the request and redirect it to the tomcat internal server which is working on port 8080

The configuration for the HTTPd service to file located at: /etc/httpd/conf.d/example.conf


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

         #	RequestHeader set X-Forwarded-Proto "https"
         #	RequestHeader set X-Forwarded-Port "443"

          LogLevel debug
          ErrorLog ${APACHE_LOG_DIR}/error.log
          CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>
        

#

##

```
dnf install httpd
systemctl start httpd
systemctl enable httpd

dnf install mod_ssl
```




```
<VirtualHost *:80>
    ServerName solr.home54.de
    Redirect permanent / https://solr.home54.de/
</VirtualHost>

 

<VirtualHost *:443>
    ServerName solr.home54.de

    ProxyPreserveHost On
    ProxyRequests On

    SSLEngine On
    SSLProxyEngine On

    ProxyPass / http://localhost:3000/
    ProxyPassReverse / http://localhost:3000/

    SSLCertificateFile    /etc/ssl/wildcard-home54.crt
    SSLCertificateKeyFile /etc/ssl/wildcard-home54.key
    SSLCACertificateFile  /etc/ssl/wildcard-home54.cc

#    ErrorLog ${APACHE_LOG_DIR}/grafana_error.log
#    CustomLog ${APACHE_LOG_DIR}/grafana_access.log combined

</VirtualHost>
```

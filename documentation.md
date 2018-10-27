# VPN

Documentation détaillée qui détaille pas à pas l'installation du service OpenVPN (configuration du firewall, génération des clés, mise en place du service OpenVPN)
* https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-16-04

# Serveur Web

## Mise en place de nginx

[Source] https://www.techrepublic.com/article/how-to-install-and-enable-modsecurity-with-nginx-on-ubuntu-server/

Pour pouvoir mettre en place un serveur Web nginx équipé d'un WAF (modsecurity pour nginx), il faut compiler les sources "à la main" et ne pas prendre le paquet nginx disponible sur les sources officielles

* Installation des dépendances nécessaires pour la compilation

```bash
apt-get install -y git build-essential libpcre3 libpcre3-dev libssl-dev libtool autoconf apache2-dev libxml2-dev libcurl4-openssl-dev automake pkgconf
```

* Compilation de ModSecurity

```bash
cd /usr/src
​git clone -b nginx_refactoring https://github.com/SpiderLabs/ModSecurity.git
cd ModSecurity
​./autogen.sh
./configure --enable-standalone-module --disable-mlogc
```

* Compilation de nginx
```bash
cd /usr/src
​wget http://nginx.org/download/nginx-1.13.4.tar.gz
tar xvzf nginx-1.13.4.tar.gz && rm nginx-1.13.4.tar.gz
cd nginx-1.13.4/
​./configure --user=www-data --group=www-data --add-module=/usr/src/ModSecurity/nginx/modsecurity --with-http_ssl_module --without-http_gzip_module
​make
​make install
sed -i "s/#user nobody;/user www-data www-data;/" /usr/local/nginx/conf/nginx.conf
```

* Vérification que la compilation s'est bien faite
```bash
/usr/local/nginx/sbin/nginx -t
```

* Création du service

* Pour pouvoir utiliser nginx comme un service, càd utiliser les commandes du type ```systemctl {command} {service}```, il faut créer un fichier dans le dossier ```/lib/systemd/system```

```bash
vim /lib/systemd/system/nginx.service
```

* Contenu du fichier nginx.service
```
[Service]
Type=forking
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/usr/local/nginx/sbin/nginx -s reload
KillStop=/usr/local/nginx/sbin/nginx -s stop

KillMode=process
Restart=on-failure
RestartSec=42s

PrivateTmp=true
LimitNOFILE=200000

[Install]
WantedBy=multi-user.target
```

* Mise à jour du système qui gère les services
```bash
systemctl daemon-reload
```

* On importe les fichiers de configuration de modsecurity
```bash
cp /usr/src/ModSecurity/modsecurity.conf-recommended /usr/local/nginx/conf/modsecurity.conf
cp /usr/src/ModSecurity/unicode.mapping /usr/local/nginx/conf/
```

* Ajouts des règles pour le WAF

* On crée le fichier ```/usr/local/nginx/conf/modsec_includes.conf```

```bash
include modsecurity.conf
include owasp-modsecurity-crs/crs-setup.conf
include owasp-modsecurity-crs/rules/*.conf
```

* On active le WAF (par défaut, il est en mode détection seulement)
```bash
sed -i "s/SecRuleEngine DetectionOnly/SecRuleEngine On/" /usr/local/nginx/conf/modsecurity.conf
```

* On télécharge les règles WAF fournies par OWASP
```bash
cd /usr/local/nginx/conf
git clone https://github.com/SpiderLabs/owasp-modsecurity-crs.git
cd owasp-modsecurity-crs
mv crs-setup.conf.example crs-setup.conf
cd rules
mv REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
mv RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf.example RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf
```

## Mise en place du site

* Installation de php

Le site installé nécessite une version de PHP >= 7.2 qui n'est pas dans les dépots officiels

```bash
add-apt-repository ppa:ondrej/php
apt-get update
apt-get install php7.2-cli php7.2 php7.2-fpm php7.2-sqlite php7.2-xml php7.2-json php7.2-zip php7.2-curl
```

* Mise en place de php-fpm

php-fpm est le service qui gère php pour un serveur nginx. Il faut donc s'assurer que celui est démarré et lancer au redémarrage. Pour s'en assurer, on lance la commande:

```bash
systemctl enable php7.2-fpm
```

* Mise en place du site

Les sources du site sont placées dans le dossier ```/var/www/site```

Pour que les fichiers .php soient interprétés par nginx, il faut rajouter les lignes suivantes:

```bash
location ~ \.php$ {
	include snippets/fastcgi-php.conf;
	fastcgi_pass unix:/run/php/php7.2-fpm.sock;
}
```

* Sécurisation des headers du site

[Source] https://observatory.mozilla.org/analyze/pa9.fr

```bash
client_max_body_size 20M;
client_body_buffer_size 20M;
send_timeout 300s;

#add_header Content-Security-Policy "default-src 'self'; script-src 'self'; img-src 'self'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com; frame-src 'self'; object-src 'self'";
add_header Strict-Transport-Security "max-age=31536000" always;
add_header X-XSS-Protection "1; mode=block";
add_header X-Content-Type-Options nosniff;
add_header X-Frame-Options "SAMEORIGIN";	 
proxy_cookie_path / "/;HTTPOnly; Secure";
```

* Mise en place du certificat SSL

[Source] https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04

* Ajout du dépot
```bash
add-apt-repository ppa:certbot/certbot
apt-get update
apt-get install python-certbot-nginx
```

* Génération des certificats
```bash
certbot --nginx -d pa9.fr
```

* Redirection des requêtes vers le site

```bash
server {
 listen 80 default_server;
 server_name _;
 return 301 https://pa9.fr$request_uri;
}

server {
 listen 443;
 server_name _;
 return 301 https://pa9.fr$request_uri;
}
```

Une copie du fichier nginx.conf se trouve dans le dépot

# Mail

* Installation des dépendances

```bash
apt-get install postfix postfix-mysql dovecot-core dovecot-imapd dovecot-lmtpd dovecot-mysql
```

* Mise en place de la BDD

* Connexion
```bash
mysql -uroot -p
```

* BDD
```sql
CREATE DATABASE mailserver;
GRANT SELECT ON mailserver.* TO 'birdy'@'localhost' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
USE mailserver;

CREATE TABLE `virtual_domains` (
	`id`  INT NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(50) NOT NULL,
	PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `virtual_users` (
	`id` INT NOT NULL AUTO_INCREMENT,
	`domain_id` INT NOT NULL,
	`password` VARCHAR(106) NOT NULL,
	`email` VARCHAR(120) NOT NULL,
	PRIMARY KEY (`id`),
	UNIQUE KEY `email` (`email`),
	FOREIGN KEY (domain_id) REFERENCES virtual_domains(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `virtual_aliases` (
	`id` INT NOT NULL AUTO_INCREMENT,
	`domain_id` INT NOT NULL,
	`source` varchar(100) NOT NULL,
	`destination` varchar(100) NOT NULL,
	PRIMARY KEY (`id`),
	FOREIGN KEY (domain_id) REFERENCES virtual_domains(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO virtual_domains(name) VALUES('pa9.fr');
INSERT INTO virtual_users(domain_id, password, email) VALUES(1, ENCRYPT('password', CONCAT('$6$', SUBSTRING(SHA(RAND()), -16))), 'jean-kevin.lavilardiere@pa9.fr');
INSERT INTO virtual_aliases(domain_id, source, destination) VALUES(1, 'contact@pa9.fr', 'jean-kevin.lavilardiere@pa9.fr');
exit
```

* Configuration de Postfix, Dovevot

Les fichiers de configuration sont dans le dépot

* Mise en place de SPF, DKIM, DMARC

[Source] https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-dkim-with-postfix-on-debian-wheezy

```
SPF (Sender Policy Framework) is a system that identifies to mail servers what hosts are allowed to send email for a given domain. Setting up SPF helps to prevent your email from being classified as spam.

DKIM (DomainKeys Identified Mail) is a system that lets your official mail servers add a signature to headers of outgoing email and identifies your domain’s public key so other mail servers can verify the signature. As with SPF, DKIM helps keep your mail from being considered spam. It also lets mail servers detect when your mail has been tampered with in transit.

DMARC (Domain Message Authentication, Reporting & Conformance) allows you to advertise to mail servers what your domain’s policies are regarding mail that fails SPF and/or DKIM validations. It additionally allows you to request reports on failed messages from receiving mail servers.
```

* Installation OpenDKIM

```bash
apt-get install opendkim opendkim-tools
```

* Fichier /etc/opendkim.conf

```
AutoRestart             Yes
AutoRestartRate         10/1h
UMask                   002
Syslog                  yes
SyslogSuccess           Yes
LogWhy                  Yes

Canonicalization        relaxed/simple

ExternalIgnoreList      refile:/etc/opendkim/TrustedHosts
InternalHosts           refile:/etc/opendkim/TrustedHosts
KeyTable                refile:/etc/opendkim/KeyTable
SigningTable            refile:/etc/opendkim/SigningTable

Mode                    sv
PidFile                 /var/run/opendkim/opendkim.pid
SignatureAlgorithm      rsa-sha256

UserID                  opendkim:opendkim

Socket                  inet:12301@localhost
```

* Fichier /etc/postfix/main.cf

```
milter_protocol = 2
milter_default_action = accept

smtpd_milters = inet:localhost:12301
non_smtpd_milters = inet:localhost:12301
```

* Fichiers de configuration OpenDKIM

```bash
mkdir -p /etc/opendkim/keys
```

** Fichier /etc/opendkim/TrustedHosts

```
mail._domainkey.pa9.fr pa9.fr:mail:/etc/opendkim/keys/pa9.fr/mail.private
```

** Fichier /etc/opendkim/SigningTable

```bash
*@pa9.fr mail._domainkey.pa9.fr
```

* Génération des clés

```
cd /etc/opendkim/keys
mkdir pa9.fr
cd pa9.fr
opendkim-genkey -s mail -d pa9.fr
chown opendkim:opendkim mail.private
```

Il faut maintenant récupérer le contenu du fichier ```mail.txt``` que l'on doit copier dans les enregistrements DNS

* Configuration de la zone DNS

* A

![alt A](https://github.com/philippekhau/pa8/blob/master/img/mail/A.PNG)

* MX

![alt MX](https://github.com/philippekhau/pa8/blob/master/img/mail/mx.PNG)

* DKIM

![alt DKIM](https://github.com/philippekhau/pa8/blob/master/img/mail/dkim.PNG)

* DMARC

![alt DMARC](https://github.com/philippekhau/pa8/blob/master/img/mail/dmarc.PNG)

* SPF

![alt SPF](https://github.com/philippekhau/pa8/blob/master/img/mail/spf.PNG)

* Configuration de SpamAssassin

cf fichier de configuration

* Ajout de règles à la main

cf fichier /etc/spamassassin/local.cf

# Centralisation des logs

[Source] https://www.grafikart.fr/tutoriels/serveur/elastic-stack-elk-980

Suite ELK:
ElasticSearch: BDD
Logstash: Traitement des fichiers de logs
Kibana: Affichage des logs dans des graphiques

* Installation de Elasticsearch

```bash
apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list
apt-get install elasticsearch
```

Fichier de config dans le dépot

* Installation des Filebeat sur les machines clientes

[Source] https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-getting-started.html

* Création des tableaux dans Kibana

- Accéder à la Web Interface http://localhost:5601
- Ajout des index dans Managements Tools
- Créer des visualisations
- Créer des tableaux de bord
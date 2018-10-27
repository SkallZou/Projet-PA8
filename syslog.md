# Installation

```bash
apt-get install syslog-ng
```

# Configuration

* désactiver les logs dans la console (/etc/default/syslog-ng)
```bash
CONSOLE_LOG_LEVEL=1
```

* autoriser la création de dossiers (/etc/syslog-ng/syslog-ng.conf)
```bash
options { create_dirs(yes); }
```

* autoriser les logs depuis l'extérieur (port IANA = 514)
```
source s_net {
  udp(port(514));
}
```

# Ajouter des logs

## Configuration du serveur

* modifier le fichier /etc/syslog-ng/syslog-ng.conf
* ajouter une destination 
```bash
destination dst-pfSense {
	file("/var/log/pfSense/$YEAR-$MONTH-$DAY/pfSense.log");
};
```
* ajouter les logs
```bash
log {
	source(s_net);
	destination(dst-pfSense);
};
```
* enregistrer les modifications
* vérifier s'il n'y a pas d'erreur
```bash
syslog-ng -t
```
* redémarrer le service
```bash
systemctl restart syslog-ng.service
```

## Création de filtres

### Filtre nginx

```bash
filter f-pfSense-nginx {
	host("pfSense.localdomain") and program("nginx");
};

filter f-pfSense-not-nginx {
	host("pfSense.localdomain") and not program("nginx");
};

destination dst-pfSense-nginx {
	file("/var/log/pfSense/$YEAR-$MONTH-$DAY/nginx.log");
};

log {
	source(s_net);
	filter(f-pfSense-nginx);
	destination(dst-pfSense-nginx);
};
```

## Configuration des clients

### Client pfSense
* Status > System logs
* Cocher "Enable Remote Logging"
![alt pfsense](https://github.com/philippekhau/pa8/blob/master/img/syslog/pfSense-client.PNG)


# Améliorations

* ajouter des filtres
* utiliser des outils de visualisation des logs
* créer des scripts de suppression de logs
* créer des scripts de sauvegarde des logs

# Sources

* https://doc.ubuntu-fr.org/syslog-ng
* http://www.opendoc.net/solutions/comment-installer-configurer-serveur-log-syslog-ng
* https://phelepjeremy.wordpress.com/2017/06/20/configuration-dun-serveur-syslog-ng/

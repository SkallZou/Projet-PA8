# Installation

L'installation de Squid est très simple et s'effectue avec la commande suivante :

    apt-get install squid
Selon la version de Squid que vous installez, il se peut que certaines configurations ne fonctionnent pas, vous pouvez vérifier avec :

    squid3 -v


# Configuration

Le fichier de configuration Squid étant très long, on le sauvegarde d'abord (**/etc/squid/squid.conf**) et on l'édite ensuite :

    cp /etc/squid/squid.conf  /etc/squid/squid.conf.old
    sudo nano /etc/squid/squid.conf

## Nommer le proxy

    visible_hostname dmz-server
![Nom du serveur](https://ezddy.github.io/hostname.png)
> **Notes** (selon les versions):
> - Il est important d'utiliser le nom de votre machine lorsque vous attribuez le hostname
> - Par défaut, les nouvelles versions détectent automatiquement le hostname
## Choisir le port
Vous pouvez choisir celui qui vous convient en remplaçant 3128 par le port de votre choix (souvent le 8080)  :

    http_port 8080
## Choisir l'interface
Par défaut le serveur proxy sera en écoute sur toutes les interfaces. Pour des raisons de sécurité, vous pouvez décider de le mettre en écoute que sur votre réseau local.
Dans notre cas, on possède une seule interface :

    http_port 192.168.2.101:8080
![IP de l'interface connectée au LAN](https://ezddy.github.io/ip_interface.png)
## Filtrage des URLs
On va installer SquidGuard qui est un addon pour Squid va filtrer les requêtes qui vont match avec des URLs.

    wget www.squidguard.com/Downloads/squidGuard-current.tar.gz
    cd squidGuard-1.4
    ./configure
    make
    sudo make install
Il faut ensuite télécharger une blacklist qui contiendra les domaines et les URLs considérés malveillants :

    wget -q http://dsi.ut-capitole.fr/blacklists/download/blacklists.tar.gz
    
On l'extrait dans le dossier db de squidGuard pour qu'il puisse générer des .db et les traiter à chaque fois qu'on essaie d'accéder à un URL puis on dit à SquidGuard de tout compiler une seule fois et pas à chaque fois qu'un URL est appelé (gain de temps et de performance)

    tar zxvf blacklists.tar.gz
    /usr/bin/squidGuard -C all -d

## Redémarrage du proxy
Il ne reste plus qu'à tester le proxy en le redémarrant :

    sudo /etc/init.d/squid restart
## Sources

[system-linux.eu](https://www.system-linux.eu/index.php?post/2010/05/31/Installation-et-Configuration-du-Proxy-Squid)

[squid-cache.eu](https://wiki.squid-cache.org/ConfigExamples)

[ubuntu-fr.org](https://doc.ubuntu-fr.org/squid)

----------





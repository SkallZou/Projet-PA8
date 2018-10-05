# Installation

L'installation de l'Active Directory est très simple, vous trouverez ci-dessous un tutoriel assez complet sur l'installation de l'AD:

https://blogs.technet.microsoft.com/canitpro/2017/02/22/step-by-step-setting-up-active-directory-in-windows-server-2016/

#### Conseil avant l'installation de l'Active Directory (pour éviter les erreurs, mais si vous suivez le tuto tout ira bien ;) )
* Désactiver DHCP sur votre serveur -> donner une adresse ip statique 
* Installer le serveur DNS avant l'installation de l'Active Directory

# Gestion des utilisateurs

L'Active Directory est un annuaire LDAP crée par Microsoft permettant : 
* Une administration centralisée et simplifiée 
  * Gestion des utilisateurs  
  * Mise en place d'une hiérarchie avec les unités d'organisations (= groupe à plusieurs niveau)
  * Gestion des stratégies de groupe 
  
* Unifier l'authentification : Ressource de l'annuaire accessible par un utilisateur authentifié.

### _Création d'un utilisateur_

Connectez-vous sur le serveur, allez ensuite dans les **outils d'administration** puis dans **Utilisateurs et ordinateurs Active Directory**.
Vous devez tomber sur une page comme celle-ci : 

![alt text](https://github.com/SkallZou/Projet-PA8/blob/master/photo/Utilisateur%20et%20ordinateur%20ad1.JPG)

Déroulez le nom de domaine de votre serveur (VGuys.fr sur mon screen), ensuite clic droit sur le dossier **Users** puis **Nouveau** et **Utilisateur**
Puis vous vous trouverez sur cette page :

![alt text](https://github.com/SkallZou/Projet-PA8/blob/master/photo/Utilisateur%20et%20ordinateur%20ad3.JPG)

Rentrez ensuite les informations de l'utilisateur (Prénom, Nom, Nom d'ouverture de session, mot de passe...)

Une fois que toutes les informations ont été saisies, appuyez sur suivant pour finaliser la création de l'utilisateur.

# Partage de fichier et Stratégie de groupe 

Les stratégies de groupe se font sur les unités d'organisation et les partages sont pour les groupes.

La procédure de création d'une unité d'organisation ou un groupe est la même que pour un utilisateur. L'unité d'organisation est un type d'objet de l'annuaire qui peut contenir d'autres objets du domaine comme des utilisateurs, groupe ou même d'autres unités d'organisations. C'est principalement l'objet de l'Active Directory qui permet de classer, d'organiser, ou de hiérarchiser les utilisateurs de l'annuaire.

![alt text](https://github.com/SkallZou/Projet-PA8/blob/master/photo/Utilisateur%20et%20ordinateur%20ad4.JPG)

Par exemple sur ce screen, on y trouve un groupe nommé **BadGuys** et plusieurs unités d'organisation (Direction, PériodeEssai)

Je vais vous montrer comment fonctionne le système de partage puis comment appliquer les stratégies de groupe sur les unités d'organisations.

### _Partage de fichiers_

Prenez n'importe quel dossier et faites clic droit, allez sur **propriétés**, puis dans l'onglet **partage** et cliquez sur **partager...** 

Et vous devez tomber sur cette fenetre : 

![alt text](https://github.com/SkallZou/Projet-PA8/blob/master/photo/Partage%20de%20fichier2.JPG)

Sur cette fenêtre, vous avez la possibilité de définir les niveaux d'autorisation (RO, WO, R/W) pour un uitilisateur ou un groupe. On peut aussi interdire l'accès et la modification pour un groupe ou un utilisateur. L'intérêt est que si jamais on avait un utilisateur qui se trouve dans plusieurs groupe, et qu'il a un groupe qui n'a pas l'autorisation de lecture et de modification alors l'utilisateur ne pourra pas non plus avoir accès au dossier.

Pour cela allez dans **partage avancé** -> **autorisation** et ajouter un groupe en cochant toutes les cases de la colonne **deny**

![alt text](https://github.com/SkallZou/Projet-PA8/blob/master/photo/Partage%20de%20fichier6.JPG)

Avec le système de partage, on peut aussi lier un dossier partagé comme une partition pour un utilisateur. Copier le **chemin réseau** qui se trouve sur l'onglet **partage**, ensuite revenez sur la fenêtre **Utilisateurs et ordinateurs Active Directory** puis allez dans les **propriétés** d'un utilisateur qui possèdent tous les droits du fichier, allez ensuite sur l'onglet **profil** et connecter une partition au dossier partagé (coller le **chemin réseau** que je vous ai demandé de copier).

![alt text](https://github.com/SkallZou/Projet-PA8/blob/master/photo/Partage%20de%20fichier4.JPG)

La prochaine fois que l'utilisateur se connecte, il verra une nouvelle partition qui correspond à son dossier.

### _Stratégie de groupe_

(A continuer...)

# Connexion d'un ordinateur sur le domaine

Pour se connecter au domaine, il faut modifier l'adresse du serveur DNS de la machine et mettre l'adresse DNS de l'Active Directory. Pour cela, allez dans les **panneaux de configuration** -> **Réseau et Internet** -> **Centre Réseau et partage** -> cliquer sur **Ethernet** -> **propriétés** -> **Protocole Internet version 4** -> entrer l'adresse du serveur DNS

![alt text](https://github.com/SkallZou/Projet-PA8/blob/master/photo/Connexion2.JPG)

Allez maintenant dans les informations systèmes de la machine : Panneau de configuration\Système et sécurité\Système

Cliquyer sur **Modifier les paramètres**

![alt text](https://github.com/SkallZou/Projet-PA8/blob/master/photo/Connexion3.JPG)

Entrer le nom de votre domaine, puis cliquer sur OK, on demandera de vous connecter avec un compte autorisé à joindre le domaine : entrer le compte administrateur du domaine.

Puis vous devez recevoir un message de bienvenue comme celui-ci :

![alt text](https://github.com/SkallZou/Projet-PA8/blob/master/photo/Connexion5.JPG)

Redemarrez la machine pour que les nouveaux paramètres soient pris en compte.

Une fois redemarrer, entrer un compte utilisateur que vous avez créé auparavant. Je me connecte avec celui qui possède la nouvelle partition crée à partir du dossier partagé.

![alt text](https://github.com/SkallZou/Projet-PA8/blob/master/photo/connexion1.JPG)

Ouvrez ensuite l'explorateur windows et on remarque qu'il a bien une nouvelle partition privée

![alt text](https://github.com/SkallZou/Projet-PA8/blob/master/photo/Partage%20de%20fichier5.JPG)

# Améliorations

* Apprendre et maitriser Powershell pour optimiser l'automatisation des stratégies de groupes avec les scripts.
* Installer DFS (Distributed Files System)












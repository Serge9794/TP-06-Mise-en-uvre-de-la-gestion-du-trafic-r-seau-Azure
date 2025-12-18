# TP-06-Mise-en-uvre-de-la-gestion-du-trafic-r-seau-Azure
La gestion du trafic réseau dans Azure repose sur une architecture multicouche qui combine des services spécialisés pour assurer la connectivité, la sécurité, l'équilibrage de charge et le routage efficace.
-------
# TP 06 – Mise en œuvre de la gestion du trafic réseau Azure


#  Objectifs pédagogiques

À l’issue de ce TP, on est capable de :

_Déployer une infrastructure Azure à l’aide d’un modèle ARM

_Configurer un Azure Load Balancer (L4)

_Mettre en œuvre une Azure Application Gateway (L7)

_Tester la répartition du trafic HTTP selon différents scénarios

#  Scénario du laboratoire

Mon organisation héberge un site web public.
Je dois  :

_Répartir la charge HTTP entre plusieurs machines virtuelles

_Fournir des contenus images et vidéos depuis des serveurs distincts

_Implémenter :

~Un équilibreur de charge Azure

~Une passerelle d’application Azure

#  Toutes les ressources seront déployées dans la même région Azure.

# _Architecture cible_

# 1 réseau virtuel

# 3 sous-réseaux

# 3 machines virtuelles

# 1 Azure Load Balancer (public)

# 1 Azure Application Gateway

🛠️ Nommage utilisé dans ce TP 
Ressource	Nom
Groupe de ressources	az104-rg6
Réseau virtuel	vnet-tp06
Load Balancer	lb-tp06
Application Gateway	appgw-tp06
Machines virtuelles	az104-06-vm0 , az104-06-vm1 , az104-06-vm2 
#  Tâche 1 – Provisionner l’infrastructure via un modèle ARM
#  _Objectif_

Déployer automatiquement :

_Un VNet

_Un NSG

_Trois machines virtuelles

#  Étapes

1-Télécharger les fichiers du laboratoire :

/Allfiles/Lab06


2-Se connecter au portail Azure :
 https://portal.azure.com

3-Rechercher Deploy a custom template

4-Sélectionner Créer votre propre modèle dans l’éditeur

Charger le fichier :

az104-06-vms-template.json


Charger ensuite les paramètres :

az104-06-vms-parameters.json


# _Compléter les champs :_

|Paramètre |	Valeur|
|Abonnement|	Votre abonnement|
|Groupe de ressources|	az104-rg6
|Mot de passe	|Mot de passe sécurisé|

Sélectionner Révision + créer → Créer

⏳ Attendre ~5 minutes.

✅ Résultat attendu :

1 VNet

3 sous-réseaux

3 VMs (1 par sous-réseau)

# Tâche 2 – Configurer un Azure Load Balancer
# _Objectif_

Répartir le trafic HTTP (port 80) entre deux machines virtuelles.

# 🔹 Création du Load Balancer

|Nom :| lb-tp06|

|Type :| Public|

|SKU :| Standard|

|Région : |identique aux VMs|

# 🔹 Configuration IP Frontend

|*Paramètre*	| *Valeur*
|Nom	|fe-tp06|
|IP publique|	pip-lb-tp06|
|Allocation	|Statique|

# 🔹 Pool Backend

|Élément	|Valeur|
|Nom	|be-tp06|
|VMs	|az104-06-vm1, az104-06-vm2|
# 🔹 Règle d’équilibrage

|Paramètre|	Valeur|
|Nom	|lbrule-tp06|
|Protocole|	TCP|
|Port|	80|
|Sonde	|TCP / 80|
|Persistance|	Aucune|

# Test du Load Balancer

Copier l’IP publique frontend

Ouvrir un navigateur :

http://<ip-publique>


Actualiser plusieurs fois

✅ Résultat attendu :

Alternance entre :

Hello World from az104-06-vm1

Hello World from az104-06-vm2

#  Tâche 3 – Configurer Azure Application Gateway
# Objectif

Mettre en place un routage HTTP basé sur le chemin :

/image/* → serveur images

/video/* → serveur vidéos

# 🔹 Création du sous-réseau dédié

|Paramètre|	Valeur|
|Nom	|subnet-appgw|
|Plage|	10.60.3.224/27|

_⚠️ Application Gateway nécessite un sous-réseau dédié (/27 minimum)._

# 🔹 Création de la passerelle

|Paramètre|	Valeur|
|Nom	|appgw-tp06|
|SKU	|Standard v2|
|Instances|	2|
|IP Frontend|	Publique|

# 🔹 Pools backend

Pool	Machines

be-app	az104-06-vm1, az104-06-vm2

be-images	az104-06-vm1

be-videos	az104-06-vm2 

# 🔹 Règles de routage par chemin

Chemin	Backend

/image/*	be-images

/video/*	be-videos

# Tests Application Gateway
http://<ip-frontend>/image/
http://<ip-frontend>/video/


# ✅ Résultat attendu :

/image/ → serveur images

/video/ → serveur vidéos



# ✅ Points clés à retenir

Azure Load Balancer = couche 4 (TCP/UDP)

Application Gateway = couche 7 (HTTP/HTTPS)

Le routage basé sur le chemin est une fonctionnalité L7

Le SKU Standard v2 est recommandé en production

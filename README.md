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





---



 ## Scénario de laboratoire

Votre organisation possède un site web public. Vous devez répartir la charge des requêtes entrantes entre différentes machines virtuelles.  
Vous devez également fournir des images et des vidéos depuis différentes machines virtuelles.

Vous prévoyez de déployer :
- Un **Azure Load Balancer**
- Une **Azure Application Gateway**

Toutes les ressources se trouvent dans la même région.

---

## Compétences professionnelles

- Utiliser un modèle pour provisionner une infrastructure  
- Configurer un équilibreur de charge Azure  
- Configurer une passerelle d’application Azure  

---

## Tâche 1 : Utiliser un modèle pour provisionner une infrastructure

Dans cette tâche,j'utiliserai un modèle pour déployer :
- Un réseau virtuel
- Un groupe de sécurité réseau
- Trois machines virtuelles

### Étapes

1. Téléchargez les fichiers de laboratoire :  
   `\Allfiles\Lab06` (modèle et paramètres)

2. Connectez-vous au portail Azure :  
   https://portal.azure.com

3. Recherchez et sélectionnez **Deploy a custom template**

4. Sélectionnez **Créer votre propre modèle dans l’éditeur**

5. Sélectionnez **Charger le fichier**  
   - `\Allfiles\Labs\06\az104-06-vms-template.json`

6. Sélectionnez **Enregistrer**

7. Sélectionnez **Modifier les paramètres** et chargez :  
   - `\Allfiles\Labs\06\az104-06-vms-parameters.json`

8. Sélectionnez **Enregistrer**

### Paramètres de déploiement

| Paramètre | Valeur |
|---------|--------|
| Abonnement | Votre abonnement Azure |
| Groupe de ressources | az104-rg6 |
| Mot de passe | Fournissez un mot de passe sécurisé |

> **Remarque :**  
> Si la taille de machine virtuelle n'est pas disponible, sélectionnez une référence disponible avec au moins 2 cœurs.

9. Sélectionnez **Révision + créer** puis **Créer**

> **Remarque :**  
> Le déploiement prend environ 5 minutes.  
> Vérifiez la présence d’un réseau virtuel avec trois sous-réseaux et une machine virtuelle par sous-réseau.

---

## Tâche 2 : Configurer un équilibreur de charge Azure

Les équilibreurs de charge Azure assurent la connectivité de couche 4 entre les ressources.

### Schéma d'architecture – Équilibreur de charge
<img width="1503" height="699" alt="az104-lab06-lb-architecture" src="https://github.com/user-attachments/assets/6946540a-4731-4673-ab76-e316673409f2" />


> L'équilibreur de charge répartit la charge entre deux machines virtuelles du même réseau virtuel.

### Création de l’équilibreur de charge

1. Recherchez **Load balancers**
2. Sélectionnez **+ Créer**

#### Paramètres principaux

| Paramètre | Valeur |
|---------|--------|
| Nom | az104-lb |
| Région | Même région que les VM |
| UGS | Standard |
| Type | Publique |
| Étage | Régional |

---

### Configuration IP frontale

| Paramètre | Valeur |
|---------|--------|
| Nom | az104-fe |
| Type d'IP | Adresse IP |
| Adresse IP publique | Créer |

#### Adresse IP publique

| Paramètre | Valeur |
|---------|--------|
| Nom | az104-lbpip |
| UGS | Standard |
| Affectation | Statique |

---

### Pool backend

| Paramètre | Valeur |
|---------|--------|
| Nom | az104-be |
| Réseau virtuel | az104-06-vnet1 |
| Machines virtuelles | az104-06-vm0, az104-06-vm1 |

---

### Règle d’équilibrage de charge

| Paramètre | Valeur |
|---------|--------|
| Nom | az104-lbrule |
| Protocole | TCP |
| Port | 80 |
| Port backend | 80 |

#### Sonde de santé

| Paramètre | Valeur |
|---------|--------|
| Nom | az104-hp |
| Protocole | TCP |
| Port | 80 |
| Intervalle | 5 |

---

### Test

- Accédez à l’adresse IP publique
- Vérifiez l’affichage :
  - `Hello World from az104-06-vm0`
  - `Hello World from az104-06-vm1`

Actualisez plusieurs fois pour observer l’alternance.

---

## Tâche 3 : Configurer une passerelle d’application Azure

Azure Application Gateway fournit un équilibrage de charge **couche 7**, le routage basé sur le chemin et la terminaison SSL.
### Schéma d'architecture – Passerelle d'application
<img width="1625" height="761" alt="az104-lab06-gw-architecture" src="https://github.com/user-attachments/assets/2fe1dddb-b921-4ca3-a923-be46c3b8a26d" />


---

### Création du sous-réseau Application Gateway

| Paramètre | Valeur |
|---------|--------|
| Nom | subnet-appgw |
| Adresse | 10.60.3.224/27 |

---

### Création de la passerelle

| Paramètre | Valeur |
|---------|--------|
| Nom | az104-appgw |
| Étage | Standard V2 |
| Instances | 2 |
| Réseau virtuel | az104-06-vnet1 |
| Sous-réseau | subnet-appgw |

---

### Pools backend

- **az104-appgwbe** → VM1 & VM2  
- **az104-imagebe** → VM1  
- **az104-videobe** → VM2  

---

### Règles de routage basées sur le chemin

| Chemin | Backend |
|------|--------|
| `/image/*` | az104-imagebe |
| `/video/*` | az104-videobe |

---

### Tests

- `http://<frontend-ip>/image/`
- `http://<frontend-ip>/video/`

---


## Points clés à retenir

* Azure Load Balancer fonctionne en **couche 4**
* Azure Application Gateway fonctionne en **couche 7**
* L’équilibrage standard est recommandé pour la production
* Le routage basé sur le chemin permet de diriger le trafic intelligemment

---

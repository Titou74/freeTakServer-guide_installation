# FreeTakServer - Guide d'installation
Aide à l'installation d'un serveur FreeTakServer (FTS) sous DEBIAN 11

**INSTALLATION RÉALISÉ SOUS DEBIAN 11 AVEC FTS 1.9.6 & PYTHON 3.9**
**LA PROCÉDURE RISQUE D'ÉVOLUER À L'AVENIR AVEC LES PROCHAINES VERSIONS DE FTS**

## Prérequis
- Dispose d'un serveur ou d'un VPS
- Avoir DEBIAN 11 installé

## Étape 1 : mise à jour du système et installation des dépendances
### Mise à jour de DEBIAN
```
sudo apt update
sudo apt upgrade
```

### Installation de python 3 et pip
```
sudo apt install python3 python3-pip
```

Vérification des versions de Python
```
python3 --version
pip --version
```

### Installation de dépendances nécessaires
```
sudo apt install python3-dev python3-setuptools build-essential python3-gevent python3-lxml libcairo2-dev
```
```
sudo pip3 install wheel pycairo WTForms==2.3.3 SQLAlchemy==1.3.20 eventlet
```

## Étape 2 : installation et paramétrage de FTS et FTS UI
### Installation de FTS
```
sudo pip install FreeTAKServer[ui]
```

### Correction du paramétrage par défaut de FTS
```
sudo nano /usr/local/lib/python3.9/dist-packages/FreeTAKServer/controllers/configuration/MainConfig.py
```

En ligne 18, remplacer
```
python_version = 'python3.8'
```
par
```
python_version = 'python3.9'
```

### Premier lancement de FTS
```
sudo python3 -m FreeTAKServer.controllers.services.FTS
```
Répondez aux quelques questions qui seront demandé afin de créer la configuration initiale (simplement faire entrer)

### Création et lancement du service FTS
```
sudo nano /etc/systemd/system/FreeTAKServer.service
```
```
[Unit]
Description=FreeTAK Server service
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
ExecStart=/usr/bin/python3 -m FreeTAKServer.controllers.services.FTS

[Install]
WantedBy=multi-user.target
```

Lancement du service
```
sudo service FreeTAKServer start
```

### Configuration de FTS UI
```
sudo nano /usr/local/lib/python3.9/dist-packages/FreeTAKServer-UI/config.py
```

Changer les lignes IP / APPIP / MAPIP par l'IP de votre serveur.
Corriger la ligne certpath pour y indiquer la version 3.9 de Python (au lieu de 3.8)

### Premier lancement de FTS UI
```
sudo python3 /usr/local/lib/python3.9/dist-packages/FreeTAKServer-UI/run.py
```

Connectez vous à [server-ip]:5000 pour vérifier le fonctionnement

### Création du service FTS UI
```
sudo nano /etc/systemd/system/FreeTAKServerUI.service
```
```
[Unit]
Description=FreeTAK Server service
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
ExecStart=/usr/bin/python3 /usr/local/lib/python3.9/dist-packages/FreeTAKServer-UI/run.py

[Install]
WantedBy=multi-user.target
```
Lancement du service
```
sudo service FreeTAKServerUI start
```

## Étape 3 : modification du compte FTS
Connectez vous à l'UI [server-ip]:5000

Connectez vous avec admin/password

Créer un nouvelle utilisateur

Déconnectez vous puis connectez vous avec le nouvelle utilisateur

Supprimer l'utilisateur admin

## Étape 4 : sécuration de la communication entre FTS et FTS UI (optionnelle)

### Générer un token/mot de passe complexe.
Vous pouvez utiliser "https://passwordsgenerator.net/", en veillant à cocher "Exclude Ambiguous Characters"

### Configurer le token dans la configuration de FTS
```
sudo nano /opt/FTSConfig.yaml
```
Décommentez la ligne "Exclude Ambiguous Characters" et ajouter ensuite le token entre guillement simple
```
  FTS_WEBSOCKET_KEY: 'votre_token_archi_complexe'
```

### Configurer le token dans FTS UI
```
sudo nano /usr/local/lib/python3.9/dist-packages/FreeTAKServer-UI/config.py
```
Saisissez votre token après "WEBSOCKETKEY"
```
    WEBSOCKETKEY = 'votre_token_archi_complexe'
```

### Redémarrer les services FTS et FTSUI
```
sudo service FreeTAKServer restart
```
```
sudo service FreeTAKServerUI restart
```

Vérifier le bon fonctionnement de FTS UI sur [server-ip]:5000
Si ça ne fonctionnement pas, vérifiez que vous n'avez pas fait d'erreur de saisie au niveau des tokens

# Porial — logiciel BIM d'exploitation

Installation Docker autonome de démonstration.

Image `ghcr.io/porial/porial` : serveur Porial complet (binaire obfusqué, interface web, lecture
des maquettes IFC) et un PostgreSQL dans le même `docker-compose.yml`.

**Licence** : logiciel propriétaire, licence d'évaluation gratuite (`LICENCE.md`). Test et
démonstration seulement ; redistribution et usage commercial interdits ; tous droits réservés au
concepteur ; fourni sans garantie.

> Configuration de **démonstration** : la sécurité est volontairement réduite (détection
> d'intrusion coupée, mots de passe sans verrouillage, cookies en clair sur HTTP, accès ouvert à
> tout le réseau local). À réserver à des données non sensibles, sur un réseau de confiance.

## Installation

Prérequis : Docker avec le plugin `compose`, machine `amd64` ou `arm64` — Linux, macOS, ou
Windows 10/11 64 bits avec Docker Desktop (moteur WSL 2, conteneurs Linux).

### Linux, macOS

Depuis un dossier vide :

```sh
mkdir porial && cd porial
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/install" \
    ghcr.io/porial/porial install --site-url http://192.168.1.20:8080
docker compose up -d
```

Remplacer `192.168.1.20` par l'adresse de la machine sur le réseau. Si le port 8080 est déjà pris
(`port is already allocated`), choisir un autre port dès l'installation :

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/install" \
    ghcr.io/porial/porial install --port 8090 --site-url http://192.168.1.20:8090
```

(`--postgres-port` fait de même pour PostgreSQL, 54320 par défaut.)

Déjà installé et `docker compose up -d` a échoué sur le port ? Ne relancez pas `install` : modifiez
`PORIAL_PORT` et `SITE_URL` dans `.env`, puis relancez `docker compose up -d`. (`install --force`
tirerait un nouveau mot de passe de base, que la base déjà créée refuserait.)

### Windows 64 bits (PowerShell)

Docker Desktop doit être démarré. Depuis un dossier vide :

```powershell
mkdir porial; cd porial
docker run --rm -v "${PWD}:/install" ghcr.io/porial/porial install --site-url http://192.168.1.20:8080
docker compose up -d
```

Avec l'invite de commandes (`cmd`), remplacer `"${PWD}:/install"` par `"%cd%:/install"`. Mêmes
options `--port` et `--postgres-port` que ci-dessus. Pour ouvrir le site aux autres postes du
réseau, autoriser Docker Desktop dans le pare-feu Windows (réseau privé) à la première demande.
Les commandes de mise à jour et de désinstallation ci-dessous sont les mêmes sous Windows.

### Ensuite

La commande `install` écrit `docker-compose.yml`, `.env`, ce fichier et `LICENCE.md`, en tirant au hasard le
mot de passe de la base, le compte administrateur, son mot de passe et le secret de
l'application ; elle les affiche à la fin. `--site-url` est l'adresse sous laquelle les postes du
réseau ouvriront le site (adresse IP ou nom de la machine).

Le premier démarrage crée le schéma de la base (quelques minutes). Suivi :
`docker compose logs -f porial`.

## Utilisation

| | |
|---|---|
| Site | `http://<machine>:8080` (`PORIAL_PORT` du `.env`) |
| Compte | `INITIAL_SUPERUSER_EMAIL` / `INITIAL_SUPERUSER_PASSWORD` du `.env` |
| PostgreSQL | port `54320` (`POSTGRES_PORT` du `.env`), base et utilisateur `porial`, mot de passe `x-postgres-password` |

Le compte administrateur n'est créé qu'au premier démarrage, tant qu'aucun superuser n'existe ;
modifier ensuite ces deux valeurs ne change plus rien. Changez son mot de passe depuis le profil.

### Sans serveur d'e-mail (défaut)

Aucun SMTP n'est configuré en démonstration et le site fonctionne sans : connexion par mot de
passe uniquement, et les comptes se créent depuis la page d'administration des utilisateurs par le
superuser (« Créer un utilisateur », avec un mot de passe). Les invitations sont enregistrées mais
aucun message ne part, ce que l'écran signale. Pour les activer, renseigner `EMAIL_HOST`… dans
`.env`, passer `ENABLE_EMAIL_CODE_LOGIN=true`, puis `docker compose up -d`.

Les ports se changent ensuite dans `.env` (`PORIAL_PORT`, `POSTGRES_PORT`, et `SITE_URL`), puis
`docker compose up -d`.

## Mise à jour

```sh
docker compose pull
docker compose up -d
```

Le schéma est mis à jour au démarrage (`INIT_SCHEMA=true`). Les données vivent dans les volumes
`porial_postgres_data` et `porial_porial_data`.

## Sauvegarde

```sh
docker compose exec -T postgres_porial pg_dump -U porial -Fc porial > porial.dump
docker run --rm -v porial_porial_data:/data -v "$PWD:/out" alpine tar czf /out/porial-data.tgz -C /data .
```

## Désinstallation

`docker compose down` arrête les services ; `docker compose down -v` supprime aussi **toutes les
données** (base et fichiers déposés), sans retour possible.

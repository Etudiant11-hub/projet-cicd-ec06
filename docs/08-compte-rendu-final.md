# 08 - Compte rendu final

## 1. Synthèse

Ce projet met en place une chaîne CI/CD complète pour Catal-Log : à chaque modification du dépôt, une image Docker Nginx contenant un site statique est automatiquement construite et testée par GitHub Actions ; sur la branche `main`, elle est ensuite publiée dans GitHub Container Registry avec un tag et un digest uniques ; enfin, un workflow déclenché manuellement permet de valider cette image dans un environnement `recette` simulé puis de la promouvoir, sans reconstruction, vers un environnement `production-simulee`. L'ensemble est documenté et complété par une orchestration légère avec Docker Compose ainsi qu'une analyse des pratiques nécessaires pour un passage en production réelle.

## 2. Fonctionnement technique

Le chemin complet est le suivant :
1. **Commit / push** sur le dépôt GitHub déclenche `01-ci.yml`.
2. **Build + test (CI)** : le workflow vérifie la présence des fichiers attendus, valide la syntaxe de `compose.yml`, construit l'image Docker à partir du `Dockerfile`, démarre un conteneur de test et vérifie en HTTP que la page d'accueil et `version.json` répondent correctement.
3. **Publication GHCR** : sur push vers `main`, `02-publish-ghcr.yml` reconstruit l'image et la publie dans GHCR avec les tags `sha-<commit>` et `latest`, en affichant le digest obtenu dans le résumé du run.
4. **Validation recette** : `03-promote.yml`, déclenché manuellement avec un tag source, télécharge l'image déjà publiée (sans rebuild), la démarre et vérifie son bon fonctionnement en HTTP dans l'environnement GitHub `recette`.
5. **Promotion production-simulee** : si la recette est validée, le même artefact est retaggé `production-simulee` et republié dans GHCR, dans l'environnement GitHub `production-simulee`, sans jamais reconstruire l'image.

## 3. Conteneurisation C12

Le `Dockerfile` part de l'image officielle `nginx:1.27-alpine` (légère et régulièrement maintenue), copie le contenu du dossier `site/` dans le répertoire servi par Nginx, ajuste les permissions, expose le port 80 et déclare un `HEALTHCHECK` qui vérifie périodiquement que le serveur répond. L'image est construite automatiquement dans `01-ci.yml`, exécutée dans un conteneur, et testée via des requêtes HTTP réelles. Les preuves du build et du test automatisé figurent dans `03-fiche-tests.md`, celles de la publication dans `04-preuve-image.md`.

## 4. Orchestration et scaling C13

`compose.yml` décrit deux services : `web` (construit depuis le `Dockerfile` local, exposé en interne sur le réseau `cicd_net`) et `tester` (basé sur `curlimages/curl`, qui dépend de `web` et exécute automatiquement des requêtes de vérification HTTP). Ce second service illustre une coordination simple de plusieurs conteneurs. Une simulation de mise à l'échelle a été réalisée avec `docker compose up -d --scale web=2` (détails et limites dans `03-fiche-tests.md` et `02-schema-chaine-cicd.md`) : elle montre que plusieurs instances peuvent cohabiter, mais sans répartiteur de charge ni haute disponibilité réelle, ce qui ne remplace en rien une orchestration de production comme Kubernetes.

## 5. Automatisation et sécurité C14

Les trois workflows GitHub Actions (`01-ci.yml`, `02-publish-ghcr.yml`, `03-promote.yml`) couvrent respectivement le contrôle/test, la publication GHCR et la promotion manuelle. Chacun déclare des permissions minimales (`contents: read`, et `packages: write` uniquement quand nécessaire). L'authentification vers GHCR repose exclusivement sur `secrets.GITHUB_TOKEN`, un jeton temporaire généré automatiquement par GitHub, sans qu'aucun secret ne soit codé en dur dans le dépôt. Le rollback s'appuie sur les tags et digests des images déjà publiées (repromotion sans rebuild), et la sauvegarde/restauration repose sur le caractère versionné du code, des workflows et de la documentation, complété par la conservation des images dans GHCR. Détails complets dans `07-securite-minimale.md`.

## 6. Production réelle

**Gestion des secrets** : aucun secret n'est stocké dans le code ; seul `GITHUB_TOKEN`, temporaire et à portée limitée, est utilisé ici. En production réelle, tout secret supplémentaire (identifiants externes, clés d'API, certificats) devrait être placé dans GitHub Secrets ou un coffre-fort dédié (Vault, AWS/Azure Secrets Manager), jamais dans le code versionné.

**Rollback** : chaque image est identifiée par un tag et un digest uniques et immuables. Revenir à une version antérieure consiste à repromouvoir, sans reconstruction, une image déjà publiée et validée, en ciblant son tag ou directement son digest pour une garantie absolue de contenu.

**Sauvegarde/restauration** : il faudrait pouvoir restaurer le dépôt GitHub (code, Dockerfile, compose.yml, site), les workflows et la documentation (déjà versionnés avec le code), les images publiées dans GHCR (idéalement répliquées vers un second registre), la configuration des environnements GitHub (règles de protection), ainsi que les preuves d'exécution.

Deux éléments complémentaires retenus : le **contrôle des vulnérabilités** (scan automatique de l'image avant publication, ex. Trivy) et la **séparation stricte des environnements** (recette et production isolées, au-delà de la simulation actuelle via les environnements GitHub).

## 7. Preuves

*(à compléter une fois tous les workflows exécutés avec succès)*

- Lien du dépôt : https://github.com/Etudiant11-hub/projet-cicd-ec06
- Run CI réussi : **à compléter**
- Run publication GHCR : **à compléter**
- Image GHCR (tag + digest) : **à compléter** — voir `04-preuve-image.md`
- Run validation recette : **à compléter** — voir `05-preuve-recette.md`
- Run promotion production-simulee : **à compléter** — voir `06-preuve-promotion.md`

## 8. Difficultés et apprentissages

*(section personnelle à rédiger après avoir réellement exécuté le projet — le formateur attend une réflexion authentique, pas un texte générique)*

Quelques pistes de réflexion à développer avec tes propres mots une fois le pipeline exécuté :
- Ce que t'a appris la distinction entre tag et digest pour la traçabilité.
- Pourquoi la promotion "sans rebuild" est une garantie importante de cohérence entre recette et production.
- Les limites concrètes que tu as observées avec Docker Compose (scaling sans load balancer, pas de haute disponibilité).
- Une difficulté technique rencontrée (ex : configuration des environnements GitHub, timing du `sleep` dans le service `tester`, permissions GITHUB_TOKEN) et comment tu l'as résolue.

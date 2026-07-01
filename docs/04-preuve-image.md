# 04 - Preuve image GHCR

## Image publiée

- Nom de l'image : ghcr.io/etudiant11-hub/projet-cicd-ec06
- Tag principal : latest **(vérifier après le run, ou indiquer `sha-<commit>` si tu préfères tracer un commit précis)**
- Digest : **à compléter** — visible dans le résumé du run du workflow `02-publish-ghcr.yml` (section "Résumé publication" du Step Summary), ou via `docker image inspect ghcr.io/etudiant11-hub/projet-cicd-ec06:latest`
- Lien GHCR ou capture : https://github.com/Etudiant11-hub/projet-cicd-ec06/pkgs/container/projet-cicd-ec06 **(à vérifier une fois l'image publiée — GHCR crée automatiquement cette page)**

## Explication

Le tag seul (ex: `latest`) est pratique à retenir mais il est mutable : il peut pointer vers un contenu différent à chaque nouvelle publication. Le digest, en revanche, est une empreinte SHA256 calculée à partir du contenu exact du manifeste de l'image : il est unique et immuable. En conservant le couple tag + digest pour chaque publication, on garantit une traçabilité exacte de ce qui a été construit et déployé à un instant donné, et on peut à tout moment revenir précisément à une version antérieure (rollback) en utilisant son digest, même si le tag associé a depuis été réattribué à une autre image.

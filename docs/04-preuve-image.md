# 04 - Preuve image GHCR

## Image publiée

- Nom de l'image : ghcr.io/etudiant11-hub/projet-cicd-ec06
- Tag principal : latest (et sha-df3e1df pour la traçabilité exacte de ce commit)
- Digest : sha256:4bd03ff07d2c153fd15847f43189350e0450d72c301fc6b024e74ef7ad3bbac5
- Lien GHCR ou capture : https://github.com/Etudiant11-hub/projet-cicd-ec06/actions/runs/28544541141

## Explication

Le tag seul (ex: `latest`) est pratique à retenir mais il est mutable : il peut pointer vers un contenu différent à chaque nouvelle publication. Le digest, en revanche, est une empreinte SHA256 calculée à partir du contenu exact du manifeste de l'image : il est unique et immuable. En conservant le couple tag + digest pour chaque publication, on garantit une traçabilité exacte de ce qui a été construit et déployé à un instant donné, et on peut à tout moment revenir précisément à une version antérieure (rollback) en utilisant son digest, même si le tag associé a depuis été réattribué à une autre image.
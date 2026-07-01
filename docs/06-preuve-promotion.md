# 06 - Preuve promotion production-simulee

## Promotion

- Workflow concerné : 03-promote.yml
- Environnement GitHub : production-simulee
- Tag source : sha-df3e1df
- Tag cible : production-simulee
- Lien du run : https://github.com/Etudiant11-hub/projet-cicd-ec06/actions/runs/28544541141

## Point essentiel

La promotion doit réutiliser une image existante. Elle ne doit pas reconstruire l'image.

## Preuve

Le job `promote-production-simulee` ne contient aucune étape `docker build` : il se limite à `docker pull` de l'image source `ghcr.io/etudiant11-hub/projet-cicd-ec06:sha-df3e1df`, puis `docker tag` vers `ghcr.io/etudiant11-hub/projet-cicd-ec06:production-simulee`, puis `docker push`. La preuve que la promotion s'est faite sans rebuild est double : d'une part l'absence de toute instruction `docker build` dans le job (visible directement dans `03-promote.yml`) ; d'autre part le fait que le digest de l'image observé en recette (`sha256:4bd03ff07d2c153fd15847f43189350e0450d72c301fc6b024e74ef7ad3bbac5`, voir `05-preuve-recette.md`) est strictement identique au digest publié initialement lors de `02-publish-ghcr.yml` (voir `04-preuve-image.md`), ce qui prouve qu'il s'agit bien du même contenu binaire, seul le tag ayant changé.

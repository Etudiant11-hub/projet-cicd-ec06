# 06 - Preuve promotion production-simulee

## Promotion

- Workflow concerné : 03-promote.yml
- Environnement GitHub : production-simulee
- Tag source : **à compléter** (identique au tag utilisé pour la validation recette)
- Tag cible : production-simulee
- Lien du run : **à compléter** — https://github.com/Etudiant11-hub/projet-cicd-ec06/actions/workflows/03-promote.yml

## Point essentiel

La promotion doit réutiliser une image existante. Elle ne doit pas reconstruire l'image.

## Preuve

Le job `promote-production-simulee` ne contient aucune étape `docker build`. Il se contente de :
1. `docker pull` de l'image source déjà publiée (identifiée par son tag) ;
2. `docker tag` pour lui attribuer localement le tag `production-simulee`, sans modifier son contenu ni son digest ;
3. `docker push` pour republier ce même artefact sous ce nouveau tag.

La preuve que la promotion s'est faite sans rebuild est double : d'une part l'absence de toute instruction `docker build` dans le job, visible directement dans `03-promote.yml` ; d'autre part le fait que le digest de l'image `production-simulee` (visible dans GHCR) est strictement identique au digest de l'image source utilisée en recette (`05-preuve-recette.md`), ce qui prouve qu'il s'agit bien du même contenu binaire, seul le tag ayant changé.

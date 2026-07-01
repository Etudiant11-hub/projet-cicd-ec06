# 05 - Preuve recette simulée

## Workflow de validation recette

- Workflow concerné : 03-promote.yml
- Environnement GitHub : recette
- Tag source validé : **à compléter** (le tag saisi lors du déclenchement manuel, ex: `latest` ou `sha-abcdef0`)
- Digest observé : **à compléter** — affiché dans le Step Summary du job `validate-recette` (étape "Résumé recette")
- Lien du run : **à compléter** — https://github.com/Etudiant11-hub/projet-cicd-ec06/actions/workflows/03-promote.yml

## Résultat

Le job `validate-recette` télécharge l'image déjà publiée sur GHCR (sans reconstruction, via `docker pull`), la démarre dans un conteneur nommé `recette` sur le runner GitHub, puis exécute des requêtes `curl` vers `http://127.0.0.1:8080/` et `http://127.0.0.1:8080/version.json`. **À compléter avec le résultat réel observé**, par exemple : "Les deux requêtes HTTP ont réussi (code 200), confirmant que l'image publiée fonctionne correctement dans l'environnement `recette` avant toute promotion vers la production simulée."

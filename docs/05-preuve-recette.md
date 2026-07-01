# 05 - Preuve recette simulée

## Workflow de validation recette

- Workflow concerné : 03-promote.yml
- Environnement GitHub : recette
- Tag source validé : sha-df3e1df
- Digest observé : sha256:4bd03ff07d2c153fd15847f43189350e0450d72c301fc6b024e74ef7ad3bbac5
- Lien du run : https://github.com/Etudiant11-hub/projet-cicd-ec06/actions/runs/28544541141

## Résultat

Le job `validate-recette` a téléchargé l'image publiée `ghcr.io/etudiant11-hub/projet-cicd-ec06:sha-df3e1df` sans reconstruction, l'a démarrée dans un conteneur nommé `recette` sur le runner GitHub, puis a exécuté avec succès des requêtes HTTP vers `http://127.0.0.1:8080/` et `http://127.0.0.1:8080/version.json`. Le digest observé (`sha256:4bd03ff07d2c153fd15847f43189350e0450d72c301fc6b024e74ef7ad3bbac5`) est strictement identique au digest publié lors de la publication GHCR initiale, confirmant qu'il s'agit bien du même artefact.

# 03 - Fiche tests

## Test automatisé GitHub Actions

- Workflow concerné : 01-ci.yml
- Lien vers le run réussi : https://github.com/Etudiant11-hub/projet-cicd-ec06/actions/workflows/01-ci.yml **(à remplacer par le lien exact du run une fois le premier push effectué)**
- Ce qui est testé : présence des fichiers attendus (Dockerfile, compose.yml, site/index.html, site/version.json, docs/08-compte-rendu-final.md), validité de la syntaxe `compose.yml`, build de l'image Docker, démarrage du conteneur et vérification HTTP de la page d'accueil et de `version.json`.
- Résultat : **à compléter après le premier run réussi** (ex : "succès, tous les tests HTTP passent, cf. capture ou lien ci-dessus").

## Test local Docker ou Docker Compose

### Situation A - Test réalisé

Docker est installé et fonctionnel sur ma machine personnelle. J'ai donc réalisé les tests suivants en local, avant de pousser le code sur GitHub.

Commandes utilisées :

```bash
docker build -t projet-cicd-nginx:local .
docker run --rm -p 8080:80 projet-cicd-nginx:local
```

Dans un autre terminal :

```bash
curl http://127.0.0.1:8080/
curl http://127.0.0.1:8080/version.json
```

Puis avec Docker Compose :

```bash
docker compose up --build
```

Résultat observé : **à compléter avec ce que tu observes réellement**, par exemple : "L'image se construit sans erreur. Le conteneur démarre et répond en HTTP 200 sur `/` avec le contenu du site Catal-Log. `version.json` est bien accessible et renvoie les informations de version. Avec `docker compose up --build`, les deux services (`web` et `tester`) démarrent ; le service `tester` exécute ses `curl` avec succès après le délai de 3 secondes (`sleep 3`) laissant le temps au service `web` de démarrer."

## Simulation de scaling

```bash
docker compose up -d --scale web=2
docker compose ps
```

Résultat observé : **à compléter**, par exemple : "Deux instances du service `web` sont démarrées simultanément sur le réseau `cicd_net`. `docker compose ps` confirme la présence de deux conteneurs `web` (ex: `projet-cicd-ec06-web-1` et `projet-cicd-ec06-web-2`) tous les deux à l'état `running`, chacun exposant le port 80 en interne. Aucun n'est directement accessible depuis l'hôte à ce stade car aucun port n'est publié (`expose` et non `ports`) : il n'y a pas de répartiteur de charge devant eux, donc scaler ne sert ici qu'à démontrer que plusieurs instances peuvent cohabiter, pas à répartir du trafic réel."

## Limites de la simulation

- Il n'y a pas de vrai répartiteur de charge (load balancer) : mettre à l'échelle avec `--scale` crée plusieurs conteneurs identiques mais rien ne distribue automatiquement le trafic entre eux.
- Il n'y a pas de haute disponibilité réelle : si l'hôte Docker tombe, tout s'arrête ; il n'y a pas de redémarrage automatique multi-nœuds ni de tolérance de panne.
- Il n'y a pas de supervision (pas de monitoring, pas d'alerting, pas de collecte de métriques ou de logs centralisée).
- Le test dépend entièrement de l'environnement local de l'étudiant : il n'est pas reproductible de façon garantie sur une autre machine sans Docker installé, contrairement au test automatisé dans GitHub Actions qui, lui, s'exécute toujours dans un environnement identique et contrôlé (runner GitHub-hosted).
- `docker compose` reste un outil de développement/démonstration : il ne gère pas le déploiement progressif (rolling update), le rollback automatique, ni la gestion de secrets avancée qu'offrirait un orchestrateur de production comme Kubernetes.

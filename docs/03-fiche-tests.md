# 03 - Fiche tests

## Test automatisé GitHub Actions

- Workflow concerné : 01-ci.yml
- Lien vers le run réussi : https://github.com/Etudiant11-hub/projet-cicd-ec06/actions/runs/28544541141
- Ce qui est testé : présence des fichiers attendus (Dockerfile, compose.yml, site/index.html, site/version.json, docs/08-compte-rendu-final.md), validité de la syntaxe `compose.yml`, build de l'image Docker, démarrage du conteneur et vérification HTTP de la page d'accueil et de `version.json`.
- Résultat : succès, le build et les deux requêtes HTTP (`/` et `/version.json`) ont réussi dès le premier run.

## Test local Docker ou Docker Compose

### Situation A - Test réalisé

Docker est installé et fonctionnel sur ma machine personnelle. J'ai réalisé les tests suivants en local avant de pousser le code sur GitHub.

Commandes utilisées :

```bash
docker build -t projet-cicd-nginx:local .
docker run --rm -p 8080:80 --name site-test -d projet-cicd-nginx:local
curl http://127.0.0.1:8080/
curl http://127.0.0.1:8080/version.json
docker stop site-test
```

Puis avec Docker Compose :

```bash
docker compose up --build
```

Résultat observé : le build s'est terminé sans erreur (`exporting to image... done`). Le conteneur démarré manuellement a répondu correctement aux deux requêtes `curl` (HTML de la page Catal-Log et contenu de `version.json`). Avec `docker compose up --build`, le service `web` a démarré normalement (logs Nginx visibles, healthcheck actif), et le service `tester` a exécuté ses propres `curl` vers `http://web/` et `http://web/version.json` avec succès : les deux réponses (HTML complet et JSON) sont apparues dans les logs, et le conteneur `tester-1` s'est terminé avec le code de sortie `0`, confirmant la réussite du test d'intégration entre les deux services.

## Simulation de scaling

```bash
docker compose up -d --scale web=2
docker compose ps
docker compose down
```

Résultat observé : `docker compose ps` a confirmé le démarrage de deux instances distinctes du service `web` (`projet-cicd-ec06-web-1` et `projet-cicd-ec06-web-2`), toutes deux à l'état `Up` avec leur `HEALTHCHECK` en cours de démarrage (`health: starting`), chacune exposant le port 80 en interne sur le réseau `cicd_net`. Cela démontre que plusieurs instances identiques du service web peuvent cohabiter sans conflit.

## Limites de la simulation

- Il n'y a pas de vrai répartiteur de charge (load balancer) : mettre à l'échelle avec `--scale` crée plusieurs conteneurs identiques mais rien ne distribue automatiquement le trafic entre eux, et aucun port n'est publié vers l'hôte (`expose` et non `ports`) pour ces conteneurs.
- Il n'y a pas de haute disponibilité réelle : si l'hôte Docker tombe, tout s'arrête ; il n'y a pas de redémarrage automatique multi-nœuds ni de tolérance de panne.
- Il n'y a pas de supervision (pas de monitoring, pas d'alerting, pas de collecte de métriques ou de logs centralisée), même si le `HEALTHCHECK` du Dockerfile donne un premier niveau basique de surveillance locale.
- Le test dépend entièrement de l'environnement local de l'étudiant : il n'est pas garanti reproductible sur une autre machine sans Docker installé, contrairement au test automatisé dans GitHub Actions qui s'exécute toujours dans un environnement identique et contrôlé (runner GitHub-hosted).
- `docker compose` reste un outil de développement/démonstration : il ne gère pas le déploiement progressif (rolling update), le rollback automatique, ni la gestion avancée des secrets qu'offrirait un orchestrateur de production comme Kubernetes.

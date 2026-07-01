# 01 - Cadrage du projet

## Identité

- Nom et prénom : Etudiant11 (anonymisé)
- Dépôt GitHub : https://github.com/Etudiant11-hub/projet-cicd-ec06
- Date de démarrage : 2026-07-01

## Objectif

Mettre en place une chaîne CI/CD permettant de construire, tester, publier et promouvoir une image Docker Nginx contenant un site web statique pour le scénario Catal-Log.

## Contraintes du projet

- Travail individuel.
- Aucune infrastructure fournie, préparée, administrée ou maintenue par le formateur.
- Pas de serveur distant, pas de SSH, pas de cloud provider imposé.
- Les traitements principaux sont exécutés dans GitHub Actions.
- Docker local ou Docker Compose sont utilisés si l'environnement personnel le permet ; sinon la limitation doit être justifiée.
- Une VM personnelle peut être utilisée si disponible ; sinon la non-utilisation doit être justifiée.

## Choix personnels

- **Dépôt** : public, sous le compte `Etudiant11-hub`, nom `projet-cicd-ec06`. Le nom du dépôt est neutre et ne contient aucune information permettant d'identifier l'auteur, conformément à la contrainte d'anonymisation.
- **Nommage des tags d'image** : `sha-<commit>` pour la traçabilité exacte de chaque build, `latest` pour la dernière version de `main`, `production-simulee` pour l'artefact promu.
- **Environnement local** : Docker est installé et fonctionnel sur ma machine. J'ai donc pu réaliser un test local réel avec `docker build` / `docker run` et avec `docker compose`, en plus des tests automatisés dans GitHub Actions (voir `03-fiche-tests.md`).
- **VM personnelle** : je ne dispose pas de VM personnelle (VirtualBox, VMware, cloud perso). Je n'ai donc pas utilisé de VM dans ce projet. Cela n'impacte pas la chaîne CI/CD car tous les traitements (build, test, publication, promotion) s'exécutent sur les runners hébergés par GitHub, sans besoin de serveur à administrer. Cette justification répond au point du socle obligatoire concernant l'utilisation ou non d'une VM personnelle.

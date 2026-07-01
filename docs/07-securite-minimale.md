# 07 - Sécurité minimale

## Permissions GitHub Actions

- 01-ci.yml : `contents: read` — ce workflow ne fait que lire le dépôt, construire et tester l'image localement sur le runner ; il n'a besoin d'aucun droit d'écriture, ni d'accès à un registre.
- 02-publish-ghcr.yml : `contents: read, packages: write` — en plus de lire le dépôt, ce workflow doit pouvoir publier une image dans GitHub Container Registry, d'où le droit `packages: write`.
- 03-promote.yml : `contents: read, packages: write` — le workflow doit pouvoir télécharger (`pull`) et republier (`push`) une image dans GHCR, donc `packages: write` est nécessaire ; `contents: read` suffit car aucune modification du dépôt n'est faite.

Ces permissions sont volontairement limitées au strict nécessaire (principe du moindre privilège) : aucun workflow n'a de droit d'écriture sur le code du dépôt (`contents: write`), ni de droit d'administration, ce qui réduit fortement l'impact possible en cas de mauvaise configuration ou de faille dans une action tierce.

## Gestion des secrets

Aucun secret (mot de passe, clé API, token personnel) n'est stocké dans le code du dépôt, car tout ce qui est versionné dans Git reste visible dans l'historique, même si on le supprime plus tard d'un commit ultérieur — surtout problématique sur un dépôt public. Un secret codé en dur pourrait aussi être récupéré par n'importe qui ayant un accès en lecture au dépôt.

Dans ce projet, l'authentification vers GHCR utilise `secrets.GITHUB_TOKEN`, un jeton temporaire généré automatiquement par GitHub pour chaque exécution de workflow. Il est injecté à l'exécution, n'est jamais stocké dans le code, expire à la fin du run, et ses permissions sont limitées à ce qui est déclaré dans le bloc `permissions:` du workflow (ici `packages: write` pour pouvoir publier/promouvoir des images).

En production réelle, il faudrait placer dans GitHub Secrets (ou un coffre-fort dédié comme HashiCorp Vault, AWS Secrets Manager, Azure Key Vault) tout élément sensible supplémentaire : identifiants de connexion à un registre externe, clés d'API tierces, certificats TLS privés, identifiants de base de données, tokens de notification (Slack, etc.).

## Rollback

Chaque image publiée est identifiée par un tag et un digest uniques. Pour revenir à une version précédente :
- si on connaît le tag `sha-<commit>` d'une version antérieure stable, il suffit de relancer `03-promote.yml` (workflow_dispatch) en indiquant ce tag comme `source_tag` : l'image correspondante est repromue vers `production-simulee`, sans reconstruction ;
- si on préfère une garantie absolue de contenu, on peut cibler directement le digest de l'image (`ghcr.io/.../image@sha256:...`) plutôt qu'un tag, car le digest ne change jamais pour un même contenu, contrairement à un tag qui peut être réattribué.

Le fait que la promotion ne fasse jamais de rebuild garantit qu'un rollback consiste uniquement à repromouvoir un artefact déjà validé, jamais à reconstruire quoi que ce soit — ce qui élimine le risque qu'un rollback introduise involontairement un changement de comportement.

## Sauvegarde / restauration

En cas de perte totale de l'environnement, il faudrait pouvoir restaurer :
- le **dépôt GitHub** lui-même (code source, Dockerfile, compose.yml, site statique) : GitHub héberge déjà une copie distante, mais un clone local à jour ou un mirror sur un second hébergeur limiterait le risque de dépendre d'un seul fournisseur ;
- les **workflows** (`.github/workflows/*.yml`), versionnés avec le reste du code, donc déjà sauvegardés avec le dépôt ;
- la **documentation** (`docs/*.md`), également versionnée dans le dépôt ;
- les **images publiées dans GHCR** : GHCR conserve les images tant qu'elles ne sont pas supprimées manuellement, mais en production il serait prudent de répliquer les images critiques vers un second registre, ou au minimum de s'assurer que chaque image peut être reconstruite à l'identique depuis le code source versionné (reproductibilité du build) ;
- la **configuration** des environnements GitHub (`recette`, `production-simulee`) : règles de protection, secrets associés — à redocumenter si perdue, car ce n'est pas versionné dans le code ;
- les **preuves** (liens de runs, captures, digests) : à conserver dans la documentation du dépôt, comme fait dans ce projet.

La restauration consiste alors à recloner le dépôt, recréer les environnements GitHub avec leurs éventuelles règles de protection, et republier ou repromouvoir l'image nécessaire à partir du code source ou d'une image encore présente dans le registre.

## Deux éléments complémentaires

- **Contrôle des vulnérabilités** : en production réelle, il serait pertinent d'ajouter un scan automatique de l'image Docker (par exemple avec `docker scout`, Trivy ou Grype) dans le workflow de CI, afin de détecter les vulnérabilités connues dans l'image de base (`nginx:1.27-alpine`) ou dans les paquets installés, avant toute publication.
- **Séparation stricte des environnements** : les environnements GitHub `recette` et `production-simulee` utilisés dans ce projet permettent déjà de simuler cette séparation (protections, approbations possibles). En production réelle, cela irait plus loin avec des comptes/registres/réseaux physiquement ou logiquement distincts entre recette et production, empêchant qu'une erreur en recette n'impacte la production.

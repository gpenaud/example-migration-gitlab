
# Test technique - Gitlab

**Client**: IKKI-League  
**Auteur**: Guillaume Penaud   
**Mail**: guillaume.penaud@gmail.com

## Introduction

Bonjour, voici le plan de migration demandé. Par convenance, j'ai choisi de ne pas intégrer les actions de communications au sein du plan en lui-même, vous les trouverez dans le document plan-communication.md. Pour connaître les rollbacks, veuillez vous référez à plan-rollback.md. Dans le répertoire docs/, vous trouverez les détails de plusieurs étapes référencés dans le plan de migration ci-dessous, ainsi qu'un listing non exhaustif des commandes nécessaires à la réalisation de cette migration.

## Notes

J'ai volontairement explicité certaines commandes simples, pour certaines parties de la migration (analyse-prealable, préparation-vm), et suis resté volontairement flou
pour d'autres (configuration-geo) ; la raison en est simplement qu'être exhaustif m'eut pris beaucoup plus de temps, et j'ai considéré que ce qui vous intéressait était la logique et l'approche globale plus que le détail des commandes. 

## Plan de migration

| Temps | Étape | VM | Details |
|-------|-------|----|---------|
| **J-15** | Créer et provisionner la VM | Target | |
| **J-14** | Créer le playbook informationnel de la VM source | | voir: analyse-prealable.md |
|          | Créer le repository de validation | | voir: validation.md |
|          | Créer la CI du repository de validation | | voir: validation.md |
|          | Créer le playbook de validation gitlab | | voir: validation.md |
| **J-7**  | Configurer Geo | Source & Target | voir: configuration-geo.md |
|          | Lancer la réplication initiale | Source |||
| **J-7 ... J-1** | Surveiller la réplication Géo | Source |||
| **T-0** | Passer gitlab en mode maintenance | Source & Target ||
|         | Snapshot de la VM | Source ||
|         | Passer gitlab en read-only | Source ||
| **T-5** | Attendre la fin de la réplication | Target ||
| **T-10** | Promotion du secondary en primary | Target ||
| **T-15** | Basculer les DNS | Target ||
| **T-20** | Validation de l'instance gitlab | Target | voir: validation.md |
| **T-35** | Création d'un snapshot | Target ||
| **T-40 ... T-120** | Montées de version | Target | voir: upgrade.md |
| **T-125** | Validation de l'instance gitlab | Target | voir: validation.md |
| **T-150** | Désactivation du mode maintenance | Target ||
| **T-155** | Communiquer auprès des équipes || voir: communication.md
| **T-160** | Création d'un snapshot | Target ||
| **J+7** | Valider la stabilité | Target ||
| **J+14** | Décommissionner l'ancien gitlab | Source ||


## Autre options

### Storage vMotion

Cette option est très pertinente aussi dans ce cas de figure, j'ai privilégié l'option Geo car c'est l'option native de gitlab-ee omnibus pour le besoin de migration. Egalement, le fait de diposer tout au long du process d'une VM source valide et utilisable est un grand avantage. Ainsi, en cas d'imprévu, seul le switch DNS est à réaliser.

Néanmoins, je dois nuancer sur un point : utiliser Storage vMotion eut évité la fenêtre de maintenance dédiée à la configuration de Géo.
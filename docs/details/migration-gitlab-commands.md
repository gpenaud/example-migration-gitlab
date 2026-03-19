
### J-30 → Installer GitLab à la même version sur la nouvelle VM
*Sur VM Destinataire (RHEL 8):* 
```
yum install gitlab-ee=SAME_VERSION
```

### J-29 → Lancer la réplication initiale
*Sur VM Source (RHEL 7):* 

On snapshote la VM (risque éventuel de crash si misconfiguration) via VMWare.

On configure Geo:
```
# /etc/gitlab/gitlab.rb
gitlab_rails['db_key_base'] = '...'
```
```
# Enregistrer le site secondaire
gitlab-ctl set-geo-primary-node
```

### Sur la VM cible — configurer comme secondaire
*Sur VM Destinataire (RHEL 8):*
```
# /etc/gitlab/gitlab.rb
external_url 'https://gitlab-new.example.com'
roles(['geo_secondary_role'])

gitlab_rails['db_host'] = 'primary-db-host'
```
```
gitlab-ctl reconfigure

# Enregistrer auprès du primaire
gitlab-rake geo:set_secondary_as_primary
```

## J-15 ~ Surveiller la réplication
*Sur VM Destinataire (RHEL 8):*
```
# Sur la cible
gitlab-rake geo:status
```

## J-1 ~ Vérifier que la réplication est quasiment à 100% (> 99%)
*Sur VM Destinataire (RHEL 8):*
```
gitlab-rake geo:status

# résultat attendu (en gros)
Health Status: Healthy
Repositories synced: 45231 / 45231
LFS objects synced: 12453 / 12453
Attachments synced: 8921 / 8921
```

## Jour J ~ Bascule

### T+0   →  Maintenance mode ON sur source
*Sur VM Source (RHEL 7):*

```
# On passe ne mdoe maintenance
gitlab-ctl deploy-page up

# On cesse les écritures
gitlab-ctl stop puma
gitlab-ctl stop sidekiq
```

### T+5   →  Attendre sync finale (geo:status)
*Sur VM Destinataire (RHEL 8):*
```
gitlab-rake geo:status
# Attendre repositories synced: 100%
```

### T+10  →  Promote cible en primaire
*Sur VM Destinataire (RHEL 8):*
```
# promotion de la cible en primaire
gitlab-ctl promote-to-primary-node
```

### T+15  →  Basculer DNS

On bascule les DNS git.entreprise.com sur la VM destinataire (RHEL 8) 

### T+20  →  Validation

```
# Vérification de la config gitlab (affiche ID plutôt que des noms de projets)
gitlab-rake gitlab:check SANITIZE=true
```

```
# Vérifie la validité du fichier de secrets utilsié pour déchiffrer la DB.
gitlab-rake gitlab:doctor:secrets
```

```
# Tests fonctionnels
# - Connexion UI
# - Clone d'un repo
# - Push d'un commit
# - Lancement d'un pipeline CI
```

### T+30  →  Maintenance mode OFF

```
gitlab-ctl deploy-page down
```
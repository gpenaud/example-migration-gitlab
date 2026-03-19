
## Jour J (Ou nouvelle fenêtre) ~ Montée de version

Déterminer l'upgrade path - https://docs.gitlab.com/update/upgrade_paths/

Pour chaque version à installer en fonction de l'upgrade path

⚠️ **Snapshot VMware à faire ABSOLUMENT avant chaque itération.**

```
apt-get install gitlab-ee=<VERSION DE LA STEP>
gitlab-ctl reconfigure
gitlab-rake db:migrate
gitlab-ctl restart

# Valider avant de passer à l'étape suivante
gitlab-rake gitlab:check SANITIZE=true
```

Valider fonctionnellement si besoin via le playbook de validation.

⚠️ **Ne jamais passer à l'étape suivante sans valider la précédente.**
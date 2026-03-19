

# Inventaire et analyse préalable

**Client**: IKKI-League  
**Auteur**: Guillaume Penaud   
**Mail**: guillaume.penaud@gmail.com

## 1.1 Les données du système d'exploitation

    cat /etc/redhat-release  # version de la distribution
    uname -a                 # architecture
    hostnamectl              # nom de la machine
    df -h                    # espace utilisé par partition
    free -h                  # RAM allouée
    nproc                    # CPU alloués

## 1.2 Les données réseaux

    IP de la machine, ainsi que son FQDN
    Les Certificats SSL/TLS de son gitlab (emplacement, date d'expiration)
    La configuration de son pare-feu (iptables, ufw...)

## 1.3 Les informations relatives à Gitlab

Sa version, pour commencer

     # version de gitlab
    gitlab-rake gitlab:env:info
    OU
    cat /opt/gitlab/version-manifest.txt | head -1 (version courte)

    # version du paquet rhel
    rpm -qi gitlab-ee

Les fichiers de configuration essentiels

    # configuration (omnibus)
    cat /etc/gitlab/gitlab.rb

    # secrets
    cat /etc/gitlab/gitlab-secrets.json

Le contexte SELinux

    # SELinux (peut impacter le comportement de gitlab post-migration)
    getenforce
    sestatus
    cat /etc/selinux/config

Les certificats

    ls -la /etc/gitlab/ssl/
    ls -la /etc/pki/tls/certs/

Date d'expiration du certificat

    openssl x509 -in /etc/gitlab/ssl/gitlab.crt -noout -dates

Vérification de la chaîne de confiance
    
    openssl verify -CAfile /etc/pki/tls/certs/ca-bundle.crt /etc/gitlab/ssl/gitlab.crt

La taille de la base PostgreSQL embarquée

    gitlab-psql -c "SELECT pg_size_pretty(pg_database_size('gitlabhq_production'));"

La version de PostgreSQL

    gitlab-psql --version

La santé de l'instance

    gitlab-rake gitlab:check

L'intégrité des repositories

    gitlab-rake gitlab:git:fsck

Le statut des migrations BDD
    
    gitlab-rake db:migrate:status

Les logs récents pour détecter des erreurs préexistantes

    journalctl -u gitlab-runsvdir --since "24 hours ago"
    tail -n 100 /var/log/gitlab/gitlab-rails/production.log
    tail -n 100 /var/log/gitlab/nginx/gitlab_error.log

Toutes ces données sont facielement récupérables à l'aide d'un petit script ansible, via ses facts, ou un simple script bash.

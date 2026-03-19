# Inventaire et analyse préalable

**Client**: IKKI-League  
**Auteur**: Guillaume Penaud   
**Mail**: guillaume.penaud@gmail.com

# Préparation

## 1. Comment provisionnez-vous la VM RHEL8 sur VMware ?

Deux options. Soit j'ai l'accès administrateur sur l'infrastructure VMWare, et dans ce cas, je génère une ressource VM via mon provider vmware sous terraform. 

Sans oublier de securiser cette VM extrêmement critique par plusieurs mécanismes cumulés ; exemple sous terraform :

    resource "vsphere_virtual_machine" "gitlab" {
        name             = "gitlab-prod"
        resource_pool_id = data.vsphere_resource_pool.pool.id
        datastore_id     = data.vsphere_datastore.datastore.id

        # mécanisme vSphere contre la suppression
        wait_for_guest_net_timeout = 0

        # mécanisme terraform natif
        lifecycle {
            prevent_destroy = true
        }
    }

Il y a d'autres façon de sécuriser (permissions vSphere, par exemple).

Sinon, je m'adresse à l'équipe infra et leur soumet la demande (avec les contraintes) une vingtaine de jours avant la date de migration. 

Ensuite, pour le provisioning, je m'appuie sur l'outil centralisé de configuration-as-code (puppet, probablement), ou le template cloud-init de la VM. 

Si rien de tout cela n'existe, j'utilise les informations récoltées dans la partie 1 pour recréer un environnement semblable. 

## 2. Quels sont les prérequis à mettre en place avant d'installer GitLab ?

Côté OS, bien veiller à être sur la version 8.10 (stable) de RHEL

Côté stockage, il faudra environ 8To de disponible ; cela peut semble énorme, mais il faut toujours compter environ 20% de plus que la quantité de data à migrer (7To+1.4To =~ 9To)

Le proxy devra être configuré à plusieurs endroits. Yum doit pouvoir accéder aux package omnibus de gitlab, donc 

    proxy=http://proxy.entreprise.com:3128
    proxy_username=USERNAME
    proxy_password=PASWORD
    proxy_auth_method=any

Mais également gitlab, et particulièrement gitlab-geo, notre outil de migration/replication (configuration liée au fait que la VM source est dans le même réseau)

    gitlab_rails['env'] = {
        'http_proxy' => 'http://proxy.entreprise.com:3128',
        'https_proxy' => 'http://proxy.entreprise.com:3128',
        # Exclure la communication interne du proxy
        'no_proxy' => '127.0.0.1,localhost,gitlab-source.entreprise.com'
    }

Côté sécurité, je me base sur la conf-as-code probablement fournie, mais reboucler avec l'équipe sécu et son responsable sera nécessaire, compte-tenu de la criticité du service.  

Mais vérifier les standards de sécurité: 

    - pas de ports inutiles ouverts
    - WAF en amont
    - serveur SSH sécurisé
    - fail2ban & crowdsec installés et configurés
    - VM derrière un bastion
    - owner du service gitlab adapté
    - config SSSD ?

## 3. Quelle version de GitLab installez-vous en premier sur la nouvelle VM, et pourquoi ?

Gitlab devra être installé à l'exacte même version que celle de la VM source (rhel7) à migrer. 

La structure des données migrées sont relatives à la version du gitlab "source". Si j'installe une version autre que celle du gitlab "source" sur ma VM de destination, les schémas de données ne matcheront pas, et le nouveau gitlab ne fonctionnera pas. 

Nous devons suivre la procédure de migration prévue par gitlab et donc partir de l'exacte même version que celle d'origine. 
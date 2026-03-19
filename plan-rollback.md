
# Plan de rollback

**Client**: IKKI-League  
**Auteur**: Guillaume Penaud   
**Mail**: guillaume.penaud@gmail.com

| Etape                     | Rollback
|:-------------------------:|:---------------------------------------:|
| Pendant configuration Geo | Backup horodaté des fichiers de configuration <br> avant toute application de modification,<br>Snapshot VMware préalable
| Pendant réplication Geo   | Rien à faire, source toujours active
| Après promote             | Re-promote la VM source, rebascule des DNS
| Pendant montée de version | Snapshot VMware avant chaque étape
| Après montée de version   | Snapshot VMware une fois migration faite

## La migration échoue à mi-chemin, il est 23h30.

    # Quelle est votre procédure exacte pour remettre l'ancienne VM en service ? 

    Comme montré ci-dessus, grâce à Geo, il n'y a rien besoin de faire, si ce n'est s'assurer que les DNS pointent bien vers la VM source, qui possède toujours toutes les données gitlab. Donc: 
        - reconfigurer les DNS vers la VM Source
        - communiquer aux équipes l'échec de ,la migration tout en els rassurant
        - retirer la banière
        - désactiver le mode maintenance de la VM source
        - réaliser un post-mortem
        - reprogrammer une migration
    
    # Quel est le délai réaliste ?

    Dans le cas d'un simple restore de snapshot VMware, le délai est extrêmement court (entre 30 secondes et 2 minutes), car il ne s'agit que de repositionner le pointeur vers le snapshot ; la taille de 7To n'entre pas en ligne de compte.
    
    # À quel moment de votre plan considérez-vous que le rollback n'est plus possible ? 

    Considérant que la VM source est toujours disponible grâce à Geo, le rollback est toujours possible, encore une fois grâce au repositionnement des DNS. Il y a éventuellement une petite fenêtre, à déterminer, en fonction de la durée auxquels les DNS se mettent à jour chez le client.  
    
    # En quoi le volume de données complique-t-il cette décision ?

    Etant sur VMWare, grâce aux fonctionnalités de snapshot, le volume de données ne complique pas réellement cette décision.
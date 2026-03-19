
# Configuration de Gitlab Geo

**Client**: IKKI-League  
**Auteur**: Guillaume Penaud   
**Mail**: guillaume.penaud@gmail.com

## Contexte: 

Pour lancer la réplication de la VM source vers la VM target via Gitlab Geo, il est nécessaire de modifier la configuration
du gitlab source. Cela peut occasionner des erreurs, et doit donc être réalisé dans une fenêtre de maintenance (plus courte),
à prévoir à J-7. 

Côté communication, un Email + Slack-Teams accompagné d'un bandeau dans gitlab le jour même me semble suffisant, en expliquant bien qu'il ne s'agit que d'une modification de configuration mineure. 

Avant toute chose, snasphot de la VM source

Puis, pour chaque bloc de configuration (3 ou 4 en tout), voici la procédure à suivre:

    - copie horodaté du fichier de configuration
    - modification de la configuration
    - validation syntaxique (via commande gitlab)
    - restart de gitlab
    - Nouveau snapshot de la VM
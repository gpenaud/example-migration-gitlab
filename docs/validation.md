# Validation de l'instance

**Client**: IKKI-League  
**Auteur**: Guillaume Penaud   
**Mail**: guillaume.penaud@gmail.com

Le plus efficace ici est de réaliser un script (ansible / bash), qui va réaliser une suite d'action communément réalisée par les développeurs ; ainsi, l'ensemble des composants (repositories, registry, runners, webhooks, tokens), seront sollicités, et donc testés au sein d'une même procédure. 

Je propose: 

- Le clone d'un repo dédié à la validation 
- Le push d'un commit
- Lancement d'un pipeline CI avec build & push d'image

On valide que: 

    - le repository existe bien (repos)
    - notre utilisateur peut bien s'y connecter et y pusher (tokens)
    - le pipeline se lance et passe (registry & runners)
    - nous recevons bien la notification (webhooks) 

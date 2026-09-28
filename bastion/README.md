# Bastion d'accès (Teleport)

Point d'accès unique et sécurisé aux serveurs. Remplace les accès SSH directs.

## Principes

- Toutes les sessions administrateur sont enregistrées
- Authentification par certificats éphémères (pas de clés statiques qui traînent)
- Plus aucun accès direct aux serveurs : tout passe par le bastion
- Couplé à un VPN mesh (Tailscale) pour l'accès distant

## Intérêt sécurité

- Traçabilité complète de qui fait quoi sur les serveurs
- Réduction de la surface d'attaque (un seul point d'entrée durci)
- Révocation d'accès simple et centralisée

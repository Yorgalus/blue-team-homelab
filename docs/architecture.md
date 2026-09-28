# Notes d'architecture

## Logique Zero Trust

Le principe : ne rien autoriser par défaut, puis ouvrir uniquement ce qui est nécessaire.

- Segmentation réseau stricte : les zones ne communiquent pas librement entre elles
- La DMZ est la seule zone exposée à Internet
- Le SIEM est le réseau le plus isolé (il détient les preuves)
- Tout accès humain passe par le WAF (HTTPS) ou le bastion (SSH)

## Ordre de déploiement typique

1. Analyse de l'existant et définition de l'architecture cible
2. WAF en DMZ (point d'entrée)
3. Bastion et segmentation réseau
4. SIEM et agents de détection
5. Règles de détection et tableaux de bord
6. Supervision (métriques, dashboards)
7. Exploitation : réduction des faux positifs, alerting, réponse à incident

## Ce que ce lab m'a appris

Comprendre comment les briques défensives s'articulent, passer de la construction à l'exploitation (traiter de vraies alertes, réduire le bruit), et le lien entre défense et offensif : savoir comment on attaque aide à mieux détecter.

# WAF / Reverse Proxy (BunkerWeb)

Un WAF open source basé sur nginx, configuré par variables d'environnement. Une seule instance protège plusieurs services, chacun avec ses propres règles.

## Principe

- Intègre l'OWASP Core Rule Set (SQLi, XSS, LFI) pour filtrer le trafic malveillant
- Déployé en DMZ comme unique point d'entrée HTTPS
- Approche progressive : mode détection d'abord (on observe), puis mode blocage (on filtre)
- Multi-services : un domaine par service interne, avec des politiques indépendantes

## Exemple de services exposés (valeurs génériques)

| Domaine (exemple)        | Service cible        | Politique                 |
|--------------------------|----------------------|---------------------------|
| bastion.lab.local        | Bastion              | CORS désactivé            |
| dashboards.lab.local     | Grafana              | Liste blanche IP interne  |
| metrics.lab.local        | Prometheus           | Limitation de débit       |
| siem.lab.local           | Interface SIEM       | Ban temporaire si suspect |

## Bonnes pratiques appliquées

- Aucun accès direct aux services internes depuis Internet
- Seul le HTTPS (443) est autorisé vers le reverse proxy
- Journalisation de tout le trafic vers le SIEM

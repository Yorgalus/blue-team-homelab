# Blue Team Home Lab

![Zero Trust](https://img.shields.io/badge/Approche-Zero%20Trust-2C3E50?style=flat-square)
![WAF](https://img.shields.io/badge/WAF-BunkerWeb-00A5A5?style=flat-square)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-005792?style=flat-square)
![Monitoring](https://img.shields.io/badge/Monitoring-Prometheus%20%2B%20Grafana-E6522C?style=flat-square)

Reproduction d'une infrastructure de sécurité et de supervision de type Zero Trust dans un home lab. L'objectif est de mettre en pratique le déploiement d'un WAF, d'un SIEM, d'un bastion d'accès et d'une stack de supervision, et de comprendre comment ces briques s'articulent.

> Toutes les adresses IP et noms de domaine de ce dépôt sont des exemples. Aucune configuration ne correspond à une infrastructure réelle. C'est un lab pédagogique monté sur des machines virtuelles.

## Architecture réseau

```mermaid
flowchart TB
    NET([Internet])

    subgraph DMZ["DMZ · 10.0.10.0/24 (zone exposée)"]
        WAF["WAF / Reverse Proxy<br/>BunkerWeb"]
    end

    subgraph INT["Réseau interne · 10.0.20.0/24"]
        BAST["Bastion<br/>Teleport"]
        GRAF["Grafana"]
        PROM["Prometheus"]
    end

    subgraph SIEM_NET["SIEM isolé · 10.0.30.0/24"]
        WAZUH["SIEM<br/>Wazuh"]
    end

    NET -->|HTTPS 443| WAF
    NET -->|SSH via VPN| BAST
    WAF --> GRAF
    WAF --> PROM
    WAF --> WAZUH
    BAST --> INT
    WAF -. logs .-> WAZUH
    BAST -. logs .-> WAZUH
    PROM --> GRAF

    classDef dmz fill:#FBEDEC,stroke:#C0392B,color:#1a1a1a
    classDef int fill:#EAF1F8,stroke:#1F6FB2,color:#1a1a1a
    classDef siem fill:#EDEAF4,stroke:#5B4B8A,color:#1a1a1a
    class WAF dmz
    class BAST,GRAF,PROM int
    class WAZUH siem
```

Principe : toute entrée passe soit par le WAF en HTTPS, soit par le bastion en SSH. La DMZ est la seule zone exposée. Le SIEM est le réseau le plus isolé, car c'est lui qui détient les preuves.

## Flux d'une requête et d'une détection

```mermaid
sequenceDiagram
    participant U as Utilisateur
    participant W as WAF (BunkerWeb)
    participant S as Service interne
    participant Z as SIEM (Wazuh)

    U->>W: Requête HTTPS
    W->>W: Filtrage OWASP CRS (SQLi, XSS, LFI)
    alt Requête malveillante
        W-->>U: Blocage
        W->>Z: Alerte (tentative d'attaque)
    else Requête légitime
        W->>S: Transmission
        S-->>U: Réponse
        W->>Z: Log d'accès
    end
    Z->>Z: Corrélation et règles de détection
```

## Briques déployées

| Rôle | Outil | Réseau (exemple) |
|------|-------|------------------|
| WAF / reverse proxy | BunkerWeb | DMZ |
| Bastion d'accès | Teleport | interne |
| SIEM / détection | Wazuh | isolé |
| Métriques | Prometheus | interne |
| Dashboards | Grafana | interne |
| VPN mesh | Tailscale | overlay |

## Contenu du dépôt

- **waf/** : configuration du WAF (reverse proxy multi-services, mode détection puis blocage)
- **siem/** : règles de détection et sources de logs surveillées
- **bastion/** : principes de l'accès distant sécurisé (sessions enregistrées, certificats éphémères)
- **monitoring/** : configuration Prometheus et dashboards Grafana
- **docs/** : notes d'architecture et démarche

## Compétences illustrées

Sécurité périmétrique (WAF, reverse proxy, bastion), segmentation réseau, supervision et détection via SIEM, tableaux de bord et alerting, démarche Zero Trust.

## Avertissement

Configurations d'exemple à but pédagogique. Les valeurs (IP, domaines, règles) sont génériques et doivent être adaptées et durcies pour tout usage réel.

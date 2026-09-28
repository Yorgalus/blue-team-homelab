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
    ADMIN([Admin distant])

    subgraph DMZ["DMZ 10.0.10.0/24"]
        WAF["WAF / Reverse Proxy<br/>BunkerWeb"]
    end

    subgraph BAS["Bastion 10.0.20.0/24"]
        TELE["Bastion<br/>Teleport"]
    end

    subgraph MON["Supervision 10.0.30.0/24"]
        GRAF["Grafana"]
        PROM["Prometheus"]
    end

    subgraph SIEMNET["SIEM isole 10.0.50.0/24"]
        WAZUH["SIEM<br/>Wazuh"]
    end

    %% Acces web : tout passe par le WAF qui reverse-proxy vers les UI
    NET -->|HTTPS 443| WAF
    WAF -->|UI| TELE
    WAF -->|UI| GRAF
    WAF -->|UI| PROM
    WAF -->|UI| WAZUH

    %% Acces admin SSH : via le bastion a travers le VPN
    ADMIN -->|SSH via VPN| TELE

    %% Collecte : Prometheus scrape, tout journalise vers le SIEM
    PROM -->|metriques| GRAF
    WAF -->|logs| WAZUH
    TELE -->|logs| WAZUH

    classDef dmz fill:#FBEDEC,stroke:#C0392B,color:#1a1a1a
    classDef bas fill:#F8EFDD,stroke:#B5710B,color:#1a1a1a
    classDef mon fill:#EAF1F8,stroke:#1F6FB2,color:#1a1a1a
    classDef siem fill:#EDEAF4,stroke:#5B4B8A,color:#1a1a1a
    class WAF dmz
    class TELE bas
    class GRAF,PROM mon
    class WAZUH siem
```

Principe : le WAF est l'unique point d'entrée web. Il fait office de reverse proxy et redirige chaque requête HTTPS vers l'interface web du service concerné (bastion, Grafana, Prometheus, SIEM), en appliquant ses règles de filtrage. L'accès administrateur en SSH passe par le bastion via le VPN. Grafana et Prometheus partagent le réseau de supervision. Le SIEM est le réseau le plus isolé, car c'est lui qui centralise les logs et les preuves.

## Flux d'une requête et d'une détection

```mermaid
sequenceDiagram
    participant U as Utilisateur
    participant W as WAF BunkerWeb
    participant S as Service interne
    participant Z as SIEM Wazuh

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
| WAF / reverse proxy (point d'entrée web) | BunkerWeb | DMZ 10.0.10.0/24 |
| Bastion d'accès SSH | Teleport | 10.0.20.0/24 |
| Métriques + dashboards | Prometheus + Grafana | Supervision 10.0.30.0/24 |
| SIEM / détection | Wazuh | Isolé 10.0.50.0/24 |
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

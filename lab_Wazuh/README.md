# Projet Lab — Réseau d'Entreprise Segmenté

> Construire l'infrastructure, l'attaquer, puis apprendre à la défendre. Ce lab est mon terrain de jeu pour cartographier le cycle de vie d'une attaque (Red) et concevoir des règles de détection sur-mesure (Blue) au cœur d'un réseau segmenté.

**Statut :** 🟡 En cours — démarré le 28 février 2026  
**Auteur :** Khalil Abdelmoula · [medium.com/@Kh_abdel]

---

## Objectif

Ce lab reproduit un environnement d'entreprise réel pour pratiquer la **défense en profondeur** et la **détection comportementale**. Kali Linux est placé dans le VLAN 20 (postes utilisateurs) pour simuler la compromission d'un poste de travail — point de départ réaliste d'un mouvement latéral vers le VLAN Serveurs.

---

## Architecture

```
                        ┌─────────────────┐
                        │  Internet / WAN  │
                        └────────┬────────┘
                                 │
                        ┌────────▼────────────────────────┐
                        │             pfSense              │
                        │   Firewall · NAT · inter-VLAN    │
                        └──┬──────┬──────┬──────┬─────────┘
                           │      │      │      │
              ┌────────────┘      │      │      └──────────┐
              │                   │      │                  │
     ┌────────▼──────┐   ┌────────▼──────────┐    ┌────────▼──────┐
     │   VLAN 10     │   │     VLAN 20        │    │   VLAN 99     │
     │   Serveurs    │   │  Postes utilisateurs│   │  Management   │
     │               │   │                    │    │               │
     │ Windows Server│   │ Poste Windows      │    │  Admin only   │
     │ Active Dir.   │   │ Sysmon + Agent     │    │  Accès pfSense│
     └───────────────┘   │                    │    └───────────────┘
              ▲           │ Kali Linux         │
              │           │ (poste compromis)  │
              │           └────────┬───────────┘
              │                    │
              └── pivot VLAN 10 ───┘ mouvement latéral
                                   │ logs Sysmon
                          ┌────────▼──────────┐
                          │     VLAN 30        │
                          │       DMZ          │
                          │  Wazuh Manager     │
                          │  SIEM · Dashboard  │
                          └────────────────────┘
```

| Composant | VLAN | Rôle |
|-----------|------|------|
| **pfSense** | — | Routeur/firewall — 4 VLANs, règles inter-VLAN strictes, NAT sortant |
| **Windows Server 2019** | 10 | Contrôleur de domaine Active Directory — domaine `lab.local` |
| **Poste Windows** | 20 | Endpoint cible — Sysmon + Wazuh agent |
| **Kali Linux** | 20 | Attaquant simulé — compromission poste utilisateur → pivot |
| **Wazuh Manager** | 30 | SIEM — réception logs, dashboard, règles de détection |
| **Admin** | 99 | Accès administration pfSense uniquement |

> ⚠️ Kali est volontairement placé dans le VLAN 20 (postes) et non en Management. Cela simule un scénario réaliste : un employé ayant cliqué sur un payload. Le mouvement latéral vers le VLAN 10 Serveurs est ainsi soumis aux règles de segmentation — rendant la détection plus intéressante.

---

## Démarche

- [x] **Phase 1** — Conception réseau (adressage, plan VLAN, matrice des flux)
- [x] **Phase 2** — Déploiement pfSense & VLANs
- [x] **Phase 3** — Déploiement Active Directory (GPO de durcissement)
- [ ] **Phase 4** — Installation Wazuh Manager + agents + intégration Sysmon
- [ ] **Phase 5** — Simulation d'attaques (Kali) → validation des alertes
- [ ] **Phase 6** — Documentation (runbooks, scripts, règles de détection)

---

## Objectifs Blue Team

- Cloisonnement strict des flux inter-VLAN
- Détection comportementale : brute-force, Pass-the-Hash, énumération AD
- Simulation mouvement latéral VLAN 20 → VLAN 10 (pivot post-compromission)
- Corrélation d'événements Sysmon → alertes Wazuh
- Règles de détection basées sur les techniques **MITRE ATT&CK**
- Documentation de runbooks de réponse à incident

---

## Stack technique

`pfSense` · `Windows Server 2019` · `Active Directory` · `Sysmon` · `Wazuh` · `Kali Linux` · `Python` · `Bash` · `PowerShell`

---

*Ce projet fait partie de mon portfolio cybersécurité. Suivi de l'avancement dans les commits.*

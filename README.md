# Projet Stormshield — CSNA Security

Travaux pratiques réalisés dans le cadre de la formation **CCNA Security** (Ynov Campus Paris), sur la prise en main et la configuration d'un firewall **Stormshield Network Security (SNS)**.

| | |
|---|---|
| **Formation** | Ynov Campus Paris |
| **Matière** | CCNA Security |
| **Référent** | Achraf EN NADI |
| **Groupe** | Ibrahima GASSAMA, Dimitri ANGELY |

---

## Environnement de lab

L'infrastructure repose sur des machines virtuelles Oracle VirtualBox :

| Composant | Rôle |
|---|---|
| **SNS (Firewall A)** | Firewall Stormshield en configuration usine (« Trainee ») |
| **Graphical Client A / B** | Postes clients Linux pour les tests de connectivité |
| **Webmail** | Serveur de messagerie du lab |

---

## TP2 — Prise en main du firewall et traces

**Objectif :** se familiariser avec l'interface d'administration, durcir les paramètres de base et configurer la journalisation.

### Vérifications préalables

1. Sur **Graphical Client A**, afficher l'adresse IP :
   ```bash
   ip a
   ```
2. Tester la connectivité vers le SNS :
   ```bash
   ping 10.0.0.254
   ```

### Configuration initiale du firewall

Connexion à l'interface web d'administration (firewall en configuration usine).

| Paramètre | Valeur |
|---|---|
| Langue | Français |
| Politique de mot de passe | 16 caractères (recommandation ANSSI) |
| Fuseau horaire | Synchronisé avec la machine hôte |

**Changement de mot de passe administrateur :**

| | |
|---|---|
| Ancien mot de passe | `admin` |
| Nouveau mot de passe | `AaZzEeRrTtYy123&*` |

- Activer l'authentification **SSH** par mot de passe.
- Modifier le mot de passe de l'utilisateur `admin`, puis se reconnecter pour valider.

### Journalisation et sauvegarde

1. Vérifier que le **stockage local des logs** est activé sur le disque dur de la VM.
2. Réallouer le quota de la catégorie **Proxy POP3** vers **Connexions réseau**, puis désactiver Proxy POP3.
3. Activer tous les autres types de logs.
4. **Sauvegarder la configuration** et la télécharger sur le poste d'administration.

> Prendre l'habitude de sauvegarder la configuration à la fin de chaque TP.

---

## TP3 — Les objets

**Objectif :** créer et organiser les objets réseau, machines, services et groupes sur le **Firewall A** pour modéliser l'infrastructure de la seconde compagnie.

### Objets machines et réseaux

| Objet | Type | Valeur |
|---|---|---|
| `firewall B` | Machine | `192.36.253.20` |
| `LAN_in_B` | Réseau | — |
| `pc_admin` | Machine | `192.168.1.2` |
| `srv_dns_priv` | Machine | `172.16.1.10` |
| `srv_web_priv` | Machine | `172.16.1.11` |
| `srv_ftp_priv` | Machine | `172.16.1.12` |
| `srv_mail_priv` | Machine | `172.16.1.13` |

### Service personnalisé

| Objet | Protocole | Port |
|---|---|---|
| `webmail` | TCP | 808 |

### Groupes et DNS

- Créer le groupe **`srv_priv`** regroupant les 4 serveurs privés (`srv_dns_priv`, `srv_web_priv`, `srv_ftp_priv`, `srv_mail_priv`).
- Remplacer les serveurs DNS par défaut (`dns1.google.com`, `dns2.google.com`) par l'objet **`Gw_default`** (`192.36.253.1`) représentant la passerelle du réseau **NatNetwork**.

### Partie bonus — Import / export d'objets

1. Exporter la base d'objets au format **`.csn`**.
2. Créer le fichier **`Nouveaux_objets.csv`** contenant les objets `srv_ftp_pub` et `srv_mail_pub`.
3. Importer le fichier CSV dans la base d'objets du firewall.

---

## TP4 — Configuration réseau

**Objectif :** configurer les interfaces, le routage et le cache DNS du firewall.

### Interfaces réseau

| Interface | Adresse IP | Masque |
|---|---|---|
| **OUT** | `192.36.253.10` | `/24` |
| **DMZ1** | `172.16.1.254` | `/24` |
| **IN** | `192.168.1.254` | `/24` |

> Schéma générique du lab : `OUT` → `192.36.253.x0/24` · `DMZ1` → `172.16.x.254/24` · `IN` → `192.168.x.254/24` (x = numéro de compagnie).

### Routage et DNS

| Paramètre | Valeur |
|---|---|
| Passerelle par défaut | `192.36.253.1` |
| Objet DNS | `Gw_default` → `192.36.253.1` |

Après modification de l'adressage IP du client, se reconnecter à l'interface d'administration :

```
https://192.168.1.254/admin
```

### Cache DNS

- Activer le **proxy cache DNS**.
- Ajouter les clients autorisés dans la **liste des clients autorisés à utiliser le cache DNS**.

---

## Documents de référence

| Fichier | Contenu |
|---|---|
| `TP2 - LAB Prise en main du firewall et traces Dimitri Ibra (1).pdf` | Compte-rendu TP2 |
| `TP3 - LAB Les objets Dimitri Ibrahima (1).pdf` | Compte-rendu TP3 |
| `TP4 - LAB Configuration réseau Dimitri Ibrahima (1).pdf` | Compte-rendu TP4 |
| `CSNA_v4_Livre_de_formation_avec_annexes_et_virtual_box_2021_12_21 7.pdf` | Livre de formation CSNA v4 |

---

## Topologie simplifiée

```
                    Internet / NatNetwork
                           │
                    [ Gw_default ]
                     192.36.253.1
                           │
              ┌────────────┴────────────┐
              │      Firewall A (SNS)    │
              │  OUT: 192.36.253.10/24   │
              ├── DMZ1: 172.16.1.254/24 ─┤── Serveurs privés (172.16.1.x)
              └── IN:  192.168.1.254/24  ─┤── LAN interne (pc_admin, clients)
                           │
                    [ Firewall B ]
                     192.36.253.20
```

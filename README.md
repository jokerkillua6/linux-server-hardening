# linux-server-hardening

# Durcissement (Hardening) d'un Serveur Linux Debian/Ubuntu

Procédure complète de sécurisation initiale d'un serveur Linux exposé sur Internet : sécurisation de l'accès distant **SSH**, filtrage réseau avec **UFW** et prévention d'intrusion avec **Fail2ban**.

---

## 1. Mise à jour & Création d'un utilisateur administrateur

```bash
# Mise à jour des paquets système
apt update && apt upgrade -y

# Création d'un utilisateur avec les droits necessaire
adduser adminsec
usermod -aG sudo adminsec
```

---

## 2. Sécurisation du service SSH (`/etc/ssh/sshd_config`)
Remplacement de l'authentification par mot de passe par une paire de clés cryptographiques (Ed25519)

1. **Depuis le poste client**, générer et copier la clé publique :
```bash
ssh-keygen -t ed25519 -C "adminsec@serveur"
ssh-copy-id adminsec@IP_DU_SERVEUR
```

2. **Sur le serveur**, modifier `/etc/ssh/sshd_config` :
```text
Port 2222  #changer le port par defaut
PermitRootLogin no      #interdire la connexion directe en tant que super-user    
PasswordAuthentication no  #desactiver la connexion par mot de passe
PubkeyAuthentication yes  #autorise uniquement l'authentification par cle cryptographique
MaxAuthTries 3   #coupe la connexion apres 3 tentatives
```

3. Redémarrer le démon SSH et vérifier la configuration :
```bash
sshd -t && systemctl restart ssh
```

---

## 3. Configuration du Pare-feu UFW (Uncomplicated Firewall)
Application du principe de moindre privilège : bloquer tout les connexion entrant par default 

```bash
apt install ufw -y

# Principe par default
ufw default deny incoming
ufw default allow outgoing

# Autorisation  des flux legitimes

ufw allow 2222/tcp comment 'SSH'
ufw allow 80/tcp comment 'HTTP'
ufw allow 443/tcp comment 'HTTPS'

# Activation et vérification
ufw enable
ufw status verbose
```

---

## 4. Protection anti-Brute-Force avec Fail2ban
Bannissement automatique des adresses IP effectuant des tentatives de connexion répétées .

```bash
apt install fail2ban -y
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

Editer le fichier /etc/fail2ban/jail.local
```ini
[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
findtime = 10m
bantime = 1h
```

Démarrer le service et vérifier l'état :
```bash
systemctl enable --now fail2ban
fail2ban-client status sshd
```

---

## 5. Vérifications finales 
```bash
# Vérifier les ports ouverts et services en écoute
ss -tulnp

# Vérifier les tentatives de connexion bloquées
grep "Failed password" /var/log/auth.log | tail -n 10
```

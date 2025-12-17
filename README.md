# Guide d'Accès Remote aux Serveurs Linux

Documentation complète pour configurer et gérer l'accès distant à des serveurs Linux depuis Windows.

## 📋 Table des matières

- [Prérequis](#prérequis)
- [Option 1: SSH (Recommandé)](#option-1-ssh-recommandé)
- [Option 2: xRDP](#option-2-xrdp)
- [Option 3: X2Go](#option-3-x2go)
- [Option 4: VNC](#option-4-vnc)
- [Gestion du Firewall UFW](#gestion-du-firewall-ufw)
- [Comparaison des solutions](#comparaison-des-solutions)
- [Dépannage](#dépannage)

---

## Prérequis

### Côté serveur Linux
- Ubuntu/Debian 20.04 ou supérieur
- Accès sudo
- Connexion réseau stable

### Côté client Windows
- Windows 10/11
- Droits d'installation de logiciels

---

## Option 1: SSH (Recommandé)

### ✅ Avantages
- Léger et rapide
- Sécurisé par défaut
- Standard de l'industrie
- Pas besoin d'interface graphique

### Installation sur Linux

```bash
# Vérifier si SSH est installé
systemctl status ssh

# Si non installé
sudo apt update
sudo apt install openssh-server

# Démarrer et activer SSH
sudo systemctl enable ssh
sudo systemctl start ssh

# Autoriser SSH dans le firewall
sudo ufw allow 22/tcp
sudo ufw reload
```

### Outils clients Windows recommandés

1. **Windows Terminal + OpenSSH** (natif, gratuit)
2. **Termius** (interface moderne, sync cloud)
3. **MobaXterm** (complet avec X11)
4. **SecureCRT** (professionnel)

### Connexion basique

```bash
ssh username@server_ip
# Exemple: ssh admin@192.168.1.100
```

### Authentification par clés (recommandé)

```bash
# Sur Windows (PowerShell)
ssh-keygen -t rsa -b 4096

# Copier la clé vers le serveur
ssh-copy-id username@server_ip

# Ou manuellement
type $env:USERPROFILE\.ssh\id_rsa.pub | ssh username@server_ip "cat >> ~/.ssh/authorized_keys"
```

---

## Option 2: xRDP

### ⚠️ À utiliser si
- Vous avez absolument besoin du client RDP Windows natif
- Vous avez besoin d'une interface graphique complète
- Ce n'est PAS un serveur de production

### Installation complète

```bash
# Installer xRDP et environnement de bureau
sudo apt update
sudo apt install xrdp

# Installer un environnement de bureau léger
sudo apt install xfce4 xfce4-goodies

# Configuration
echo "xfce4-session" > ~/.xsession

# Redémarrer xRDP
sudo systemctl enable xrdp
sudo systemctl restart xrdp

# Autoriser dans le firewall
sudo ufw allow 3389/tcp
sudo ufw reload

# Vérifier le statut
sudo systemctl status xrdp
```

### Connexion depuis Windows

1. Ouvrir `mstsc.exe` (Remote Desktop Connection)
2. Entrer l'IP du serveur Linux
3. Se connecter avec identifiants Linux
4. Choisir "Xorg" comme session

### ❌ Inconvénients
- Performance inférieure à X2Go
- Problèmes de compatibilité possibles
- Consommation de ressources élevée
- Pas recommandé en production

---

## Option 3: X2Go (Meilleure alternative graphique)

### ✅ Avantages
- Meilleure performance que xRDP
- Compression efficace
- Support de la suspension de session
- Idéal pour connexions à faible bande passante

### Installation sur Linux

```bash
# Ajouter le repository X2Go
sudo add-apt-repository ppa:x2go/stable
sudo apt update

# Installer X2Go server
sudo apt install x2goserver x2goserver-xsession

# Installer un environnement de bureau (si nécessaire)
sudo apt install xfce4 xfce4-goodies

# Le port SSH (22) est utilisé, pas besoin d'ouvrir un nouveau port
```

### Installation client Windows

1. Télécharger X2Go Client: https://wiki.x2go.org/doku.php/download:start
2. Installer le client
3. Créer une nouvelle session:
   - Host: IP du serveur
   - Login: username Linux
   - Session type: XFCE (ou autre DE installé)

---

## Option 4: VNC

### Installation TigerVNC

```bash
# Installer TigerVNC server
sudo apt update
sudo apt install tigervnc-standalone-server tigervnc-common

# Configurer le mot de passe VNC
vncpasswd

# Démarrer le serveur VNC
vncserver :1 -geometry 1920x1080 -depth 24

# Autoriser dans le firewall
sudo ufw allow 5901/tcp
```

### Client Windows
- TigerVNC Viewer
- RealVNC Viewer
- TightVNC

---

## Gestion du Firewall UFW

### Commandes essentielles

```bash
# Vérifier le statut
sudo ufw status
sudo ufw status verbose
sudo ufw status numbered

# Autoriser un port
sudo ufw allow 22/tcp          # SSH
sudo ufw allow 3389/tcp        # RDP
sudo ufw allow 5432/tcp        # PostgreSQL

# Autoriser depuis une IP spécifique
sudo ufw allow from 192.168.1.100 to any port 22

# Autoriser depuis un sous-réseau
sudo ufw allow from 192.168.1.0/24 to any port 5432

# Lister les ports autorisés
sudo ufw status | grep ALLOW

# Supprimer une règle
sudo ufw status numbered
sudo ufw delete [numéro]
# Ou
sudo ufw delete allow 3389/tcp

# Activer/Désactiver UFW
sudo ufw enable
sudo ufw disable

# Réinitialiser toutes les règles
sudo ufw reset
```

### Ports standards

| Service | Port | Commande |
|---------|------|----------|
| SSH | 22 | `sudo ufw allow 22/tcp` |
| RDP (xRDP) | 3389 | `sudo ufw allow 3389/tcp` |
| VNC | 5900-5910 | `sudo ufw allow 5901/tcp` |
| PostgreSQL | 5432 | `sudo ufw allow 5432/tcp` |
| MySQL | 3306 | `sudo ufw allow 3306/tcp` |
| HTTP | 80 | `sudo ufw allow 80/tcp` |
| HTTPS | 443 | `sudo ufw allow 443/tcp` |

---

## Comparaison des solutions

| Critère | SSH | xRDP | X2Go | VNC |
|---------|-----|------|------|-----|
| **Performance** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **Sécurité** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| **Facilité** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **Ressources** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **GUI** | ❌ | ✅ | ✅ | ✅ |
| **Production** | ✅ | ❌ | ⚠️ | ⚠️ |

### Recommandations par cas d'usage

| Besoin | Solution recommandée |
|--------|---------------------|
| Administration serveur | **SSH** |
| Gestion BDD (PostgreSQL, MySQL) | **SSH** |
| Application GUI occasionnelle | **X2Go** |
| GUI haute performance | **NoMachine** |
| Client RDP Windows obligatoire | **xRDP** |
| Développement | **SSH + VS Code Remote** |

---

## Dépannage

### SSH ne fonctionne pas

```bash
# Vérifier le service
sudo systemctl status ssh

# Vérifier les logs
sudo journalctl -u ssh

# Vérifier la configuration
sudo sshd -t

# Redémarrer SSH
sudo systemctl restart ssh

# Vérifier le firewall
sudo ufw status | grep 22
```

### xRDP écran noir

```bash
# Vérifier les logs
sudo tail -f /var/log/xrdp.log
sudo tail -f /var/log/xrdp-sesman.log

# Reconfigurer la session
echo "xfce4-session" > ~/.xsession

# Redémarrer xRDP
sudo systemctl restart xrdp
```

### Problèmes de connexion réseau

```bash
# Vérifier l'IP du serveur
ip addr show

# Tester la connectivité depuis Windows
ping server_ip
telnet server_ip 22

# Vérifier les ports en écoute
sudo netstat -tlnp | grep -E '22|3389|5901'
# Ou
sudo ss -tlnp | grep -E '22|3389|5901'
```

### UFW bloque la connexion

```bash
# Désactiver temporairement pour tester
sudo ufw disable

# Voir les règles refusées dans les logs
sudo tail -f /var/log/ufw.log

# Réactiver avec la bonne règle
sudo ufw enable
sudo ufw allow 22/tcp
```

---

## Sécurité - Bonnes pratiques

### SSH

```bash
# Désactiver connexion root
sudo nano /etc/ssh/sshd_config
# PermitRootLogin no

# Changer le port SSH (optionnel)
# Port 2222

# Activer uniquement authentification par clés
# PasswordAuthentication no

# Redémarrer SSH
sudo systemctl restart ssh
```

### Fail2Ban (protection brute force)

```bash
# Installer Fail2Ban
sudo apt install fail2ban

# Configurer
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

# Activer
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Vérifier les bans
sudo fail2ban-client status sshd
```

---

## Ressources supplémentaires

- [OpenSSH Documentation](https://www.openssh.com/manual.html)
- [xRDP Official Site](http://www.xrdp.org/)
- [X2Go Wiki](https://wiki.x2go.org/)
- [UFW Documentation](https://help.ubuntu.com/community/UFW)

---

## Contribution

Ce guide est maintenu pour faciliter l'administration des serveurs Linux. N'hésitez pas à contribuer avec des améliorations ou corrections.

## Licence

MIT License - Libre d'utilisation et modification
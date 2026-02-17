# 🔗 Connecter GitHub au LFS Gitea

## 📦 Prérequis

Git LFS doit être installé sur la machine.

### Installer Git LFS

**Windows**

```bash
# 1. Ce rendre sur : https://git-lfs.com/
# 2. Cliquer sur le boutton "Download"
# 3. Installer le .exe télécharger précédemment
```

**Mac**

```bash
brew install git-lfs
```

**Linux**

```bash
sudo apt install git-lfs
```

<br />

### Activer Git LFS (à faire une seule fois par machine)

```bash
git lfs install
```

Cette commande configure Git pour utiliser LFS globalement.

<br /><br />

---

<br /><br />

## 📁 Ajouter la config LFS externe dans le repo GitHub

Dans ton dépôt GitHub, crée le fichier :

```
.lfsconfig
```

Avec :

```ini
[lfs]
url = https://gitea.crzgames.com/CrzGamesOrga/AetherRoyale-Client.git/info/lfs
```

Ce fichier indique à Git d’envoyer les fichiers LFS vers Gitea au lieu de GitHub.

<br /><br />

---

<br /><br />

## 🎮 Tracker les fichiers Unreal

Dans ton projet :

```bash
git lfs track "*.uasset"
git lfs track "*.umap"
git lfs track "*.fbx"
git lfs track "*.wav"
```

Cela va créer/modifier automatiquement :

```
.gitattributes
```

<br /><br />

---

<br /><br />

## 💾 Commit de la configuration

```bash
git add .gitattributes .lfsconfig
git commit -m "Configure external LFS via Gitea"
git push
```

<br /><br />

---

<br /><br />

## 🔐 Première authentification Gitea

Lors du premier push contenant des fichiers LFS, Git demandera :

```
Username:
Password:
```

### Quelle méthode utiliser ?

Deux options sont possibles :

**Option 1 — Mot de passe Gitea**

* Username : ton utilisateur Gitea
* Password : ton mot de passe Gitea

**Option 2 — Personal Access Token (RECOMMANDÉ)**

* Username : ton utilisateur Gitea
* Password : un Personal Access Token (PAT)

### Pourquoi utiliser un PAT ?

C’est la méthode recommandée car :

* plus sécurisé que le mot de passe
* peut être révoqué à tout moment
* évite d’exposer ton vrai mot de passe
* idéal pour :

  * CI/CD
  * machines de build
  * scripts automatisés
  * Unreal build servers

### Créer un Personal Access Token (PAT) dans Gitea

1. Aller sur :

```
https://gitea.crzgames.com/user/settings/applications
```

2. Donner un nom, par exemple :

```
LFS Access
```

3. Dans `Accès aux Organisations et Dépôts`, sélectionner :

```
Tout (public, privé et limité)
```

4. Dans les permissions, cocher au minimum :

```
repository → Lecture et écriture
```

5. Cliquer sur le boutton :

```
Générer un jeton
```

6. Copier le token (il ne sera plus affiché).

### Utilisation avec Git LFS

Quand Git demande :

```
Username:
```

→ ton user Gitea

```
Password:
```

→ colle le PAT

<br /><br />

---

<br /><br />

## 📡 Ce qui se passe ensuite

Quand tu pushes :

* GitHub stocke :

  * le code
  * les pointeurs LFS

* Gitea reçoit :

  * les fichiers lourds

* OVH S3 stocke :

  * les fichiers LFS réels

Architecture finale :

```
GitHub  -> pointeurs LFS
Gitea   -> serveur LFS
OVH S3  -> stockage réel
```

<br /><br />

---

<br /><br />

## ⚠️ Important pour les autres développeurs

Chaque développeur doit faire une seule fois :

```bash
git lfs install
```

Sinon :

* les assets Unreal ne seront pas téléchargés correctement
* ils verront des fichiers texte au lieu des vrais fichiers

<br /><br />

---

<br /><br />

## 🔎 Vérifier que ça fonctionne

```bash
git lfs ls-files
```

Si les `.uasset`, `.umap`, etc. apparaissent → LFS fonctionne correctement.

<br /><br />

---

<br /><br />

## 💡 Si des assets ont déjà été commit AVANT LFS

Il faut les migrer :

```bash
git lfs migrate import --include="*.uasset,*.umap,*.fbx,*.wav"
```

Sinon ils resteront stockés directement dans GitHub.

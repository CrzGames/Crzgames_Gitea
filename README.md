# Crzgames – Gitea (Kubernetes) – Serveur LFS pour GitHub

Ce projet permet de déployer **Gitea sur Kubernetes** afin de l’utiliser **uniquement comme Proxy Git LFS**, tout en continuant à utiliser **GitHub comme forge principale** (code, PR, Issues, CI/CD).

L’objectif est de séparer clairement :

```
GitHub → code source
Gitea → endpoint LFS
OVH S3 → stockage réel des fichiers LFS
```

Cela permet :

* d’éviter les coûts GitHub LFS
* de versionner des projets Unreal Engine (assets lourds)
* de garder GitHub pour le développement

<br /><br />

---

<br /><br />

# 🧱 Architecture

```
GitHub (repo principal)
        ↓
Git LFS client
        ↓
Gitea (endpoint LFS)
        ↓
OVH Object Storage (S3)
```

Gitea ne sert ici **que de passerelle LFS**.
Le code reste sur GitHub.

<br /><br />

---

<br /><br />

# 📦 Structure du dépôt

```
Crzgames_Gitea/
  k8s/
    values.yaml
    secret-s3.yaml
```

Il n’y a volontairement que 2 fichiers :

* `values.yaml` → configuration Helm Gitea
* `secret-s3.yaml` → credentials OVH S3

<br /><br />

---

<br /><br />

# 🧰 Prérequis

Ton cluster doit déjà avoir :

* Ingress Controller NGINX
* cert-manager
* Un ClusterIssuer configuré :

```
letsencrypt-production
```

* Un DNS configuré :

```
gitea.crzgames.com → IP du LoadBalancer NginxIngressController
```

<br /><br />

---

<br /><br />

# 🔐 1) Configurer le secret OVH S3

Éditer :

```
k8s/secret-s3.yaml
```

Et remplacer :

```
REPLACE_ME
```

par tes vraies clés OVH :

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: gitea-ovh-s3
  namespace: gitea
type: Opaque
stringData:
  accessKey: "REPLACE_ME"
  secretKey: "REPLACE_ME"
```

⚠️ IMPORTANT
Ne commit jamais les vraies clés dans un repo public.

<br /><br />

---

<br /><br />

# 🚀 2) Installer Gitea avec Helm

Ajouter le repo Helm officiel :

```bash
helm repo add gitea-charts https://dl.gitea.com/charts/
helm repo update
```

Installer / mettre à jour Gitea :

```bash
kubectl create namespace gitea
kubectl apply -f secret-s3.yaml
helm upgrade --install gitea gitea-charts/gitea -n gitea -f values.yaml
```

Helm va automatiquement :

* créer le namespace `gitea`
* déployer Gitea
* configurer l’Ingress HTTPS
* activer le serveur LFS
* connecter le stockage LFS à OVH S3

<br /><br />

---

<br /><br />

# 🌐 3) Vérifier que Gitea fonctionne

Accéder à :

```
https://gitea.crzgames.com
```

Healthcheck :

```
https://gitea.crzgames.com/api/healthz
```

Une réponse HTTP 200 signifie que Gitea est opérationnel.

<br /><br />

---

<br /><br />

# 🧠 4) Utilisation : Gitea uniquement pour le LFS

Tu vas continuer à utiliser :

* GitHub → repo principal
* Gitea → stockage LFS uniquement

### Étape importante

Dans Gitea, crée :

1. Une organisation (ex : `CrzGames`)
2. Un dépôt vide

Il est recommandé d’utiliser **le même nom que sur GitHub** pour s’y retrouver facilement.

Exemple :

| GitHub                       | Gitea                        |
| ---------------------------- | ---------------------------- |
| CrzGames/AetherRoyale-Client | CrzGames/AetherRoyale-Client |

Ce dépôt Gitea ne sert qu’à héberger les fichiers LFS.

Il peut rester vide côté code.

# Architecture Notes: Machines Virtuelles vs Conteneurs

## Tableau comparatif

| Sujet | Machines Virtuelles (VM) | Conteneurs |
|-------|--------------------------|------------|
| Partage du noyau | Chaque VM a son propre OS et noyau | Partagent le noyau de l’hôte |
| Temps de démarrage | Lent (boot complet de l’OS) | Rapide (quelques secondes) |
| Consommation de ressources | Élevée (OS complet inclus) | Faible (processus légers) |
| Isolation | Isolation forte (virtualisation matérielle) | Isolation au niveau processus |
| Complexité | Gestion plus lourde (patch OS, maintenance) | Plus simple avec Kubernetes |
| Portabilité | Images volumineuses | Images légères et portables |
| Performance | Léger overhead | Proche du natif |

---

## Quand préférer une VM ?

- Isolation forte requise
- Besoin de plusieurs systèmes d’exploitation différents
- Contraintes réglementaires strictes
- Applications legacy dépendantes d’un OS complet

---

## Quand combiner VM et conteneurs ?

- Kubernetes dans le cloud (les nodes sont des VM)
- Isolation infrastructure + flexibilité applicative
- Séparation claire entre équipes infra et applicatives

Architecture typique :

Infrastructure Cloud → Machines Virtuelles → Kubernetes → Conteneurs

---

# Recréation d’un Pod

Si un pod est supprimé, le **Deployment** le recrée automatiquement.

Pourquoi ?
Parce qu’il garantit que le nombre de replicas définis est respecté.

Si aucun nœud n’a assez de ressources :
- Les pods restent en état *Pending*
- Ils démarrent dès qu’un nœud devient disponible
- En cloud avec autoscaling, de nouveaux nœuds peuvent être créés

---

# Kubernetes et Virtualisation

## Que trouve-t-on sous k3s ?

- Un système Linux
- Un runtime de conteneur (containerd)
- Une machine virtuelle ou physique

Kubernetes ne remplace pas la virtualisation.

Les VM isolent des systèmes complets.  
Kubernetes orchestre des conteneurs.

---

## Dans le cloud

Les nodes Kubernetes sont des **machines virtuelles**.

Pile classique :

Hardware → Hyperviseur → VM → Linux → Kubernetes → Pods

---

# Architecture de Production

## Cluster

- Minimum 3 nœuds
- Plusieurs replicas de l’application
- Haute disponibilité

---

## Base de données

- PostgreSQL avec volume persistant
- Sauvegardes régulières vers stockage externe

---

## Monitoring

- Prometheus
- Grafana
- Alertes automatiques

---

## Logging

- Centralisation des logs (Loki ou ELK)

---

## CI/CD

- Build image
- Push vers registry
- Déploiement automatique (GitOps)

---

## Components Placement

| Component | Runs in Kubernetes? | Runs in VMs? | Runs outside the cluster? |
|-----------|-------------------|--------------|--------------------------|
| Application (quote-app) | ✅ Yes, in Pods (Deployment) | ❌ | ❌ |
| Database (PostgreSQL) | ✅ StatefulSet with PVC | ❌ | ❌ |
| Monitoring stack (Prometheus/Grafana) | ✅ in Pods | ❌ | ❌ |
| Logging stack (ELK/Loki) | ✅ in Pods | ❌ | ❌ |
| CI/CD runners | ❌ optional | ✅ VM or container | ❌ |
| Backup storage | ❌ | ❌ | ✅ Object storage (S3, GCS, NFS) |
| Cluster nodes / OS / kernel | ❌ | ✅ VMs or physical servers | ❌ |

## Répartition

### Dans Kubernetes
- Application
- Monitoring
- Logging

### Dans des VM
- Les nodes Kubernetes

### Hors cluster
- Registry
- Stockage de sauvegarde
- Outils CI/CD

---

# Panne Contrôlée

## Erreur introduite

Image invalide :

image: quote-app:v3

## État observé

Pod en :

ImagePullBackOff

## Événements

- Failed to pull image
- BackOff pulling image

## Correction

Remettre une image valide puis :

kubectl apply -f deployment.yaml

Le pod repasse en état Running.

---

# Stratégie de Rollout

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
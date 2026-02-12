# STEP 3 : Compare containers and virtual machines

## When would you prefer a VM over a container?

- When strong security isolation is required  
- When running different operating systems on the same host  
- For legacy applications needing full OS control  
- When strict compliance or sandboxing is necessary  

---

## When would you combine both?

- Run containers inside VMs in cloud environments  
- Add an extra security layer (VM boundary + containers)  
- Multi-tenant infrastructure where VMs isolate teams and containers run apps  
- Kubernetes clusters often run on VM nodes


# STEP 5 : Simulate failure


## Qui recrée le pod ?
Le **Deployment Kubernetes**.

## Pourquoi ?
Parce qu’il doit maintenir le nombre de pods demandé.  
Si un pod est supprimé, il en recrée un automatiquement.

## Si le nœud tombe ?
Kubernetes recrée le pod sur un autre nœud disponible.

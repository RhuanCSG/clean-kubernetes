# Fase 05 — Storage

**Camada:** armazenamento | **Estimativa:** ~2 semanas | **Ambiente:** minikube

Containers são efêmeros por design — quando um Pod morre, os dados dentro do filesystem do container somem. Esta fase cobre como o Kubernetes abstrai armazenamento persistente.

---

## O que você vai aprender

- A diferença entre PersistentVolume (o disco) e PersistentVolumeClaim (o pedido)
- Como a StorageClass automatiza o provisionamento de volumes
- Quando usar emptyDir vs. hostPath vs. PVC
- Por que um PVC em Pending é quase sempre problema de StorageClass

---

## O modelo mental do storage

```
Administrador provisiona → PersistentVolume (o disco real)
Usuário pede            → PersistentVolumeClaim (a requisição)
Kubernetes vincula      → PVC ↔ PV (binding)
Pod usa                 → volume referenciando o PVC
```

Com StorageClass, o provisionamento do PV é automático — o Kubernetes cria o PV quando o PVC é criado.

---

## Tópicos desta fase

1. **[PV e PVC](pv-pvc.md)** — ciclo de vida do armazenamento persistente
2. **[StorageClass](storage-class.md)** — provisionamento dinâmico e automático
3. **[Volumes Temporários](temp-volumes.md)** — emptyDir e hostPath para casos específicos

---

## Lab da fase

Criar PVC, montar em um Pod, escrever dados, deletar o Pod e confirmar que os dados persistem no Pod recriado.

Ver: `phases/05-storage/labs/lab.md` no repositório.

---

## Pronto quando

- [ ] Criar PVC, vinculá-lo a um Pod e confirmar persistência de dados após delete do Pod
- [ ] Resolver o cenário `01-pvc-pending` sem ajuda
- [ ] Explicar a diferença entre provisionamento estático e dinâmico
- [ ] Explicar o que acontece com o PV quando o PVC é deletado (reclaimPolicy)

---

## Próxima fase

➡️ [Fase 06 — RBAC](../fase-06-rbac/index.md)

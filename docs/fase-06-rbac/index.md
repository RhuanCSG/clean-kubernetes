# Fase 06 — Controle de Acesso (RBAC)

**Camada:** segurança | **Estimativa:** ~2 semanas | **Ambiente:** minikube

RBAC (Role-Based Access Control) é o mecanismo de autorização do Kubernetes. Ele controla quem pode fazer o quê no cluster — tanto usuários humanos quanto aplicações rodando em Pods.

---

## O que você vai aprender

- Como Pods se autenticam na API do Kubernetes usando ServiceAccounts
- A diferença entre Role (namespace) e ClusterRole (cluster inteiro)
- Como RoleBindings e ClusterRoleBindings vinculam permissões a identidades
- Como usar `kubectl auth can-i` para investigar problemas de autorização

---

## O modelo RBAC

```
Quem?          Pode fazer o quê?     Em quais recursos?
Subject     →  Role/ClusterRole   →  Resources + Verbs
(SA/User)      (permissões)          (pods, configmaps, get, list...)
        ↖ vinculados por RoleBinding ↗
```

---

## Tópicos desta fase

1. **[ServiceAccount](service-account.md)** — identidade de um Pod na API do Kubernetes
2. **[Role e ClusterRole](roles.md)** — conjuntos de permissões
3. **[Bindings](bindings.md)** — vinculando permissões a identidades

---

## Lab da fase

Criar ServiceAccount com permissão de leitura apenas, rodar um Pod com essa SA e confirmar via `kubectl auth can-i` que as permissões estão corretas.

Ver: `phases/06-rbac/labs/lab.md` no repositório.

---

## Pronto quando

- [ ] Criar SA com permissão mínima e verificar com `kubectl auth can-i`
- [ ] Resolver o cenário `01-403-forbidden` sem ajuda
- [ ] Explicar a diferença entre Role e ClusterRole
- [ ] Investigar um `403 Forbidden` e identificar qual verbo/recurso está faltando

---

## Próxima fase

➡️ [Fase 07 — Scheduling & Recursos](../fase-07-scheduling/index.md)

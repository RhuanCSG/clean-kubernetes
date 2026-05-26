# Fase 04 — Configuração & Segredos

**Camada:** config | **Estimativa:** ~2 semanas | **Ambiente:** minikube

Separar configuração do código é um princípio central de aplicações cloud-native. Esta fase cobre como o Kubernetes armazena e injeta configuração em containers.

---

## O que você vai aprender

- A diferença entre ConfigMap (configuração não-sensível) e Secret (dados sensíveis)
- As três formas de consumir configuração em um Pod: env, envFrom e volume
- O comportamento de propagação automática de ConfigMaps montados como volume
- Por que Secrets são base64, não criptografia

---

## Por que não usar variáveis de ambiente no YAML do Pod?

```yaml
# Funciona, mas:
env:
  - name: DATABASE_URL
    value: "postgres://admin:senha123@db:5432/app"
```

- A URL de banco de dados (com senha) fica exposta no YAML commitado
- Mudar a configuração requer recriar o Pod
- Não há controle de acesso por configuração

ConfigMap e Secret resolvem esses problemas: a configuração fica em objetos separados, com controle de acesso via RBAC, e pode ser atualizada sem recriar o Pod.

---

## Tópicos desta fase

1. **[ConfigMap](configmap.md)** — configuração não-sensível como strings e arquivos
2. **[Secret](secret.md)** — dados sensíveis em base64 com RBAC separado
3. **[Injeção em Pods](injection.md)** — as três formas de consumir no container

---

## Lab da fase

Criar um ConfigMap com arquivo de configuração, montá-lo como volume e observar a propagação automática ao editar o ConfigMap.

Ver: `phases/04-config-secrets/labs/lab.md` no repositório.

---

## Pronto quando

- [ ] Injetar configuração via `envFrom` e confirmar com `kubectl exec -- env`
- [ ] Montar ConfigMap como volume e ler o arquivo dentro do container
- [ ] Resolver o cenário `01-wrong-ref` sem ajuda
- [ ] Explicar por que variáveis de ambiente não são atualizadas automaticamente quando o ConfigMap muda

---

## Próxima fase

➡️ [Fase 05 — Storage](../fase-05-storage/index.md)

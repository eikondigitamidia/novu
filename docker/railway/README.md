# Novu no Railway (self-hosted)

Como este fork está rodando no Railway (deploy de 2026-09-26, Novu 3.19.0).
Baseado em `docker/community/docker-compose.yml`, trocando o docker-compose por 6 serviços Railway.

## Serviços

| Serviço   | Imagem                                   | Porta | Domínio público | Volume                |
|-----------|------------------------------------------|-------|-----------------|-----------------------|
| mongodb   | `mongo:8.0.17`                           | 27017 | não             | `/data/db` (5 GB)     |
| redis     | `redis:alpine`                           | 6379  | não             | `/data` (5 GB)        |
| api       | `ghcr.io/novuhq/novu/api:3.19.0`         | 3000  | sim             | —                     |
| worker    | `ghcr.io/novuhq/novu/worker:3.19.0`      | 3004  | não             | —                     |
| ws        | `ghcr.io/novuhq/novu/ws:3.19.0`          | 3002  | sim             | —                     |
| dashboard | `ghcr.io/novuhq/novu/dashboard:3.19.0`   | 4000  | sim             | —                     |

Todos na mesma região (sfo) e conversando pela rede privada `*.railway.internal`.

## Passo a passo

1. Criar o projeto no Railway.
2. Criar os 6 serviços acima a partir das imagens Docker (um por vez).
3. **mongodb**: start command `docker-entrypoint.sh mongod --ipv6 --bind_ip_all`
   (a rede privada do Railway é IPv6; sem `--ipv6` os outros serviços não conectam).
   Vars: `MONGO_INITDB_ROOT_USERNAME`, `MONGO_INITDB_ROOT_PASSWORD`. Volume em `/data/db`.
4. **redis**: volume em `/data`. Sem vars.
5. Gerar domínios públicos para `api` (porta 3000), `ws` (3002) e `dashboard` (4000).
6. Gerar segredos:
   ```bash
   openssl rand -hex 32   # JWT_SECRET
   openssl rand -hex 16   # STORE_ENCRYPTION_KEY (exatamente 32 chars)
   openssl rand -hex 32   # NOVU_SECRET_KEY
   ```
7. Setar as variáveis de `.env.railway.example` em cada serviço, trocando os domínios pelos gerados.
8. Redeploy de `api` depois de setar `FRONT_BASE_URL`.
9. Abrir o dashboard e criar a conta admin (primeiro acesso).

## Verificação

```bash
curl https://<api>/v1/health-check   # db e workflowQueue devem estar "up"
curl https://<ws>/v1/health-check
```

## Atualizar versão

Trocar a tag `3.19.0` nas 4 imagens Novu (api, worker, ws, dashboard) ao mesmo tempo e redeployar.

## Não configurado

- S3 (`S3_BUCKET_NAME`, `S3_REGION`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`): upload de logo/branding fica desativado até configurar.
- Provedores de email/SMS/push: configurar dentro do dashboard (Integrations).

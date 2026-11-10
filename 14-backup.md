# 14 — Backup criptografado

## O que um backup precisa ter

Três coisas: **receita** (os arquivos de compose e configuração), **segredos** (`.env`) e **dados** (banco e volumes). Sem os três, a restauração fica pela metade.

Princípio: **backup não testado não é backup.**

---

## O que entra

| Item | Como |
|---|---|
| Compose e configs (Caddy, monitoramento, n8n) | `tar` de arquivos |
| Certificados e estado do Caddy | `tar` (pastas criadas pelo root do container) |
| Volumes do Grafana e do Prometheus | container temporário `alpine` |
| Banco do n8n | `pg_dump` |
| Volume do n8n | container temporário `alpine` |

Tudo vai para uma pasta temporária, é empacotado e **criptografado** (AES-256) antes de ir para `~/backups`.

---

## Trechos importantes do script

Rode o script **sem `sudo`**, com caminhos absolutos. Com `sudo`, o `HOME` vira `/root` e o `tar` não acha os arquivos. Use `sudo` só no trecho que precisa (pastas do Caddy criadas pelo root) e depois ajuste o dono com `chown`.

**Pasta temporária, apagada no fim:**

```bash
TMP=$(mktemp -d)
trap 'rm -rf "$TMP"' EXIT
```

**Dump do banco do n8n:**

```bash
docker exec n8n-postgres pg_dump -U n8n -d n8n -Fc > "$TMP/n8n-db.dump"
```

- `pg_dump` tira uma foto consistente do banco ligado. Copiar a pasta do Postgres rodando pode corromper.
- `-Fc` é o formato custom: compactado e restaurável com `pg_restore`.
- **Não use `-t`** no `docker exec` ao gerar binário. O terminal altera quebras de linha e estraga o dump.

**Volume em modo somente leitura:**

```bash
docker run --rm -v n8n_n8n_data:/dados:ro -v "$TMP":/backup alpine \
  tar -czf /backup/n8n-data.tgz -C /dados .
```

- `:ro` é menor privilégio: o backup só lê.
- O nome do volume é `<projeto>_<volume>`. Um nome errado cria um volume novo e vazio, sem erro. Confira antes com `docker volume ls`.

**Criptografar:**

```bash
tar -czf - -C "$TMP" . | openssl enc -aes-256-cbc -pbkdf2 -salt \
  -out ~/backups/backup-$(date +%F).tar.gz.enc
```

O `openssl` pede a passphrase no terminal. Ela **nunca** fica no script nem junto do arquivo. Digite duas vezes igual: se divergir, aparece `Verify failure`.

---

## Testar a restauração (sem descriptografar em disco)

Liste o conteúdo:

```bash
openssl enc -d -aes-256-cbc -pbkdf2 -in ~/backups/backup-AAAA-MM-DD.tar.gz.enc | tar -tzvf -
```

Valide o dump do banco sem tocar no banco real:

```bash
openssl enc -d -aes-256-cbc -pbkdf2 -in ~/backups/backup-AAAA-MM-DD.tar.gz.enc \
  | tar -xzOf - ./n8n-db.dump | docker exec -i n8n-postgres pg_restore -l | head -20
```

Se aparecer `Format: CUSTOM` e uma lista de entradas, o dump está íntegro.

O `n8n-data.tgz` com poucos bytes é normal quando o n8n usa Postgres: os dados ficam no banco.

---

## Cópia para fora da VPS (regra 3-2-1)

Três cópias, em dois tipos de mídia, com uma fora do servidor. Aqui: original na VPS, cópia no PC, e snapshot do provedor.

No PC:

```bash
sftp -o ServerAliveInterval=15 vps
sftp> get backups/backup-AAAA-MM-DD.tar.gz.enc
```

Confira a integridade comparando o hash dos dois lados:

```bash
sha256sum ~/backups/backup-AAAA-MM-DD.tar.gz.enc          # na VPS (Linux)
Get-FileHash -Algorithm SHA256 .\backup-AAAA-MM-DD.tar.gz.enc   # no PC (PowerShell)
```

Maiúsculas e minúsculas não importam. Guarde o backup fora de pastas sincronizadas, e a passphrase em outro lugar.

---

## Erros que aconteceram

- **`tar` não achou os arquivos:** o script rodou com `sudo` e o `HOME` virou `/root`. Rode sem `sudo`, com caminhos absolutos.
- **Permissão negada nas pastas do Caddy:** elas são criadas pelo root do container. Use `sudo tar` só nesse trecho e `chown` no resultado.
- **`Verify failure`:** a passphrase digitada na confirmação foi diferente. Repita.
- **Dump corrompido:** o `docker exec -t` mexeu no binário. Tire o `-t`.
- **Volume vazio sem erro:** nome do volume errado. Veja `docker volume ls`.
- **`scp` parou em 85% (`Connection reset`):** antes de culpar o Fail2ban, confira `sudo fail2ban-client status sshd` e o log. Aqui não havia IP banido: era a rede. Use `sftp` com `reget` para continuar de onde parou, e confira sempre o hash, porque o `scp` deixa um arquivo parcial.

---

## Lacunas conhecidas

Este backup ainda **não** cobre `/etc` (sshd, Fail2ban, UFW), a configuração do CrowdSec e os scripts de `~/projetos`. Numa VPS nova, essa parte teria de ser refeita de memória. Também falta rotação (apagar os antigos) e automação, que depende de decidir como guardar a passphrase.

---

## ✅ Resultado esperado

- Backup criptografado em `~/backups`
- Cópia fora do servidor, com hash conferido
- Restauração testada com `pg_restore -l`

---

## ➡️ Voltar ao início

[Índice do repositório](./README.md)

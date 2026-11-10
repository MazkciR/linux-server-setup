# 08 — Docker: o básico para o servidor

## Por que usar Docker aqui?

Em vez de instalar Caddy, banco e outras aplicações direto no sistema, cada uma roda num container. O servidor fica mais organizado, dá para recriar tudo do zero e, se algo der errado, o estrago fica isolado.

---

## Quatro conceitos que aparecem o tempo todo

| Conceito | Analogia | O que é de fato |
|---|---|---|
| **Imagem** | Receita | Modelo somente leitura, com o sistema e o programa |
| **Container** | Prato feito a partir da receita | Instância da imagem rodando, com uma camada de escrita própria |
| **Volume** | Gaveta gerenciada pelo Docker | Dados que sobrevivem se o container for apagado |
| **Bind mount** | Pasta do servidor ligada ao container | Uma pasta real do host aparece dentro do container |

Se o container for removido, o que estava só na camada de escrita some. Volume e bind mount são os jeitos de guardar o que precisa ficar.

---

## Docker Compose: o arquivo que descreve tudo

O `docker-compose.yml` declara serviços, portas, volumes e redes. Você sobe tudo com um comando:

```bash
docker compose up -d
```

Comandos úteis:

```bash
docker compose ps              # o que está rodando
docker compose logs -f caddy   # acompanhar o log de um serviço
docker compose config          # valida o arquivo sem subir nada
docker compose restart caddy   # reinicia um serviço
```

---

## Cuidado: Docker passa por cima do UFW

Quando você publica uma porta com `ports:`, o Docker mexe no iptables diretamente. A porta fica aberta para o mundo mesmo que o UFW diga que não.

Para serviços internos, prenda a porta ao loopback:

```yaml
ports:
  - "127.0.0.1:5678:5678"   # só o próprio servidor acessa
```

Para conferir o que está escutando de fato:

```bash
sudo ss -tlnp
```

Depois, teste de fora do servidor (no seu PC):

```bash
nmap -Pn -p- IP_DA_VPS
```

O esperado é aparecerem só as portas que você realmente quer públicas (22, 80 e 443).

---

## Redes: três camadas

Containers só conversam se estiverem na mesma rede. Dá para usar isso para isolar:

| Rede | Quem fica nela | Característica |
|---|---|---|
| pública | Caddy | Recebe tráfego da internet |
| app | Aplicações | Tem saída para a internet |
| dados | Banco de dados | `internal: true`: sem saída e sem porta publicada |

```yaml
networks:
  app:
  dados:
    internal: true
```

Teste para confirmar que a rede interna está isolada (deve falhar):

```bash
docker compose exec postgres wget -q -T 5 -O- https://example.com
```

---

## Segredos

Senhas e chaves ficam num arquivo `.env` com permissão restrita, nunca dentro do `docker-compose.yml` nem no Git:

```bash
umask 077
touch .env
```

Gere valores aleatórios na própria VPS, por exemplo com `openssl rand -hex 32`.

---

## Erros que aconteceram

- **Container reiniciando em loop (`Restarting (1)`):** o Caddyfile tinha dois blocos globais. Para achar o erro real, filtre o ruído: `docker logs caddy 2>&1 | grep -v '"level":"info"'`.
- **Log não gravava:** o volume de logs não estava montado no container.
- **Volume vazio sem erro:** o Compose coloca o nome do projeto na frente do volume (`projeto_volume`). Um nome errado cria um volume novo e vazio, sem avisar. Confira com `docker volume ls`.

---

## ✅ Resultado esperado

- Você sabe a diferença entre imagem, container, volume e bind mount
- `docker compose ps` mostra os serviços como `running`
- Portas internas presas em `127.0.0.1` e banco em rede `internal`

---

## ➡️ Próximo passo

[09 — Caddy como servidor web](./09-caddy.md)

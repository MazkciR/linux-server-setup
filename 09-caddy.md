# 09 — Caddy: servidor web com HTTPS automático

## Por que Caddy?

O Caddy gera e renova o certificado HTTPS sozinho, via Let's Encrypt. O Caddyfile fica curto e você não precisa configurar certificado na mão.

Pré-requisito: o domínio precisa apontar para o IP da VPS (registro A) antes de subir o Caddy. Confira com:

```bash
nslookup seudominio.com.br 8.8.8.8
nslookup seudominio.com.br 1.1.1.1
```

---

## 1. Estrutura de pastas

```
~/caddy/
├── docker-compose.yml
├── Caddyfile
├── sites/
│   ├── arboritech/          ← arquivos públicos do site
│   └── pedagogia/
├── logs/                    ← log de acesso (o CrowdSec lê daqui)
├── data/                    ← certificados
└── config/
```

Os sites entram no container só para leitura (`:ro`).

---

## 2. docker-compose.yml

```yaml
services:
  caddy:
    image: caddy:2
    container_name: caddy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./sites:/srv:ro
      - ./logs:/var/log/caddy
      - ./data:/data
      - ./config:/config
```

---

## 3. Caddyfile (versão simplificada para estudo)

```
arboritech.com.br, www.arboritech.com.br {
	log {
		output file /var/log/caddy/access.log {
			mode 0644
		}
		format json
	}
	header Cache-Control "no-cache"

	@ocultos path_regexp ocultos (^|/)\.
	respond @ocultos 404

	root * /srv/arboritech
	file_server
}

pedagogiaraiodeluz.com.br, www.pedagogiaraiodeluz.com.br {
	log {
		output file /var/log/caddy/access.log {
			mode 0644
		}
		format json
	}
	header Cache-Control "no-cache"

	@ocultos path_regexp ocultos (^|/)\.
	respond @ocultos 404

	root * /srv/pedagogia
	file_server
}
```

O que cada parte faz:

- **`log`:** grava os acessos em JSON. O CrowdSec lê esse arquivo. O `mode 0644` evita o arquivo nascer só legível pelo root.
- **`Cache-Control "no-cache"`:** o navegador busca a versão nova do site sem ficar preso ao cache.
- **`@ocultos path_regexp ocultos (^|/)\.`:** pega qualquer caminho com um nome começando em ponto, como `/.git/config` e `/.env`.
- **`respond @ocultos 404`:** responde "não existe", sem entregar nada.
- **`root * /srv/...`:** pasta do site dentro do container.

---

## 4. Validar antes de subir

Validar antes evita o container entrar em loop de reinício:

```bash
docker run --rm -v ~/caddy/Caddyfile:/etc/caddy/Caddyfile caddy:latest caddy validate --config /etc/caddy/Caddyfile
```

Depois, suba ou reinicie:

```bash
cd ~/caddy
docker compose up -d
docker restart caddy
```

---

## 5. Testar

```bash
curl -sI https://arboritech.com.br/ | head -1              # esperado: HTTP/2 200
curl -sI https://arboritech.com.br/.git/config | head -1   # esperado: HTTP/2 404
curl -sI https://arboritech.com.br/.env | head -1          # esperado: HTTP/2 404
```

O teste de `/.git/config` é o mais importante. Se ele retornar 200, o histórico do seu projeto está público. Confira sempre também que o caminho que deve ser público continua em 200.

---

## Erros que aconteceram

- **Caddy em loop de reinício:** o Caddyfile tinha dois blocos globais e um `log` apontando para uma pasta sem volume montado. Solução: um único bloco global e o volume de logs no compose.
- **Log com permissão 600:** o arquivo nasceu só legível pelo root e o CrowdSec não conseguia ler. Solução: `mode 0644` no `output file`.
- **Certificado não emitido:** o DNS ainda não apontava para a VPS. Confira o DNS antes.
- **Site demorava a atualizar:** era cache do navegador. O `Cache-Control "no-cache"` resolve.

---

## ✅ Resultado esperado

- HTTPS funcionando nos dois domínios
- `/.git/config` e `/.env` retornam 404
- Você valida o Caddyfile antes de reiniciar

---

## ➡️ Próximo passo

[10 — Deploy dos sites com Git](./10-deploy-sites.md)

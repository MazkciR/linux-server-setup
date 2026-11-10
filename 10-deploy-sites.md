# 10 — Deploy dos sites com Git

## O princípio

**Pasta servida = pasta pública.** Tudo que o Caddy enxerga pode ser baixado por qualquer pessoa. Por isso cada site tem dois lugares no servidor:

| Pasta | Função | O Caddy enxerga? |
|---|---|---|
| `~/projetos/<site>` | Clone do repositório, com `.git` | **Não** |
| `~/caddy/sites/<site>` | Cópia só dos arquivos públicos | **Sim** |

O código-fonte e o histórico ficam fora da pasta pública.

---

## 1. Clonar o repositório

No servidor, com um token de leitura do GitHub:

```bash
mkdir -p ~/projetos
cd ~/projetos
git clone https://github.com/SEU_USUARIO/SEU_REPOSITORIO.git meu-site
```

Use um token **fine-grained** só com `Contents: Read-only` e só para esse repositório. Digite-o no prompt do terminal. Não coloque na URL, não grave em arquivo e não cole em chat.

---

## 2. Script de deploy

Crie `~/projetos/deploy-meusite.sh`:

```bash
#!/bin/bash
set -e

cd ~/projetos/meu-site && git pull
rsync -a --delete --exclude '.git' ./ ~/caddy/sites/meusite/
```

- `set -e`: se um passo falhar, o script para e não copia uma versão quebrada.
- `git pull`: baixa a versão mais nova do GitHub.
- `rsync -a --delete --exclude '.git'`: copia para a pasta pública e ignora o `.git`. O `--delete` remove da pasta pública o que sumiu do repositório.

Dê permissão e rode:

```bash
chmod +x ~/projetos/deploy-meusite.sh
~/projetos/deploy-meusite.sh
```

Atenção ao `--delete`: ele apaga no destino tudo que não existe na origem. Confira o caminho de destino antes de rodar, para não apagar outro site.

---

## 3. Antes de publicar um site novo

- Nenhum `.env`, chave ou token no repositório (`git ls-files | grep -iE 'env|key|secret|token'`).
- Nenhum texto de exemplo visível, como telefone ou endereço de teste (`grep -n 'XXXX\|Exemplo' index.html`).
- Repositório privado no GitHub.

Depois de publicar, teste:

```bash
curl -sI https://seudominio.com.br/ | head -1              # HTTP/2 200
curl -sI https://seudominio.com.br/.git/config | head -1   # HTTP/2 404
```

---

## Regras que ficam

- **Nunca edite a pasta pública.** O próximo deploy apaga o que foi mexido ali.
- **O GitHub é a fonte da verdade.** As pastas do servidor podem ser recriadas a partir dele.
- **Os sites não entram no backup.** São reproduzíveis pelo deploy. Já os scripts em `~/projetos` precisam entrar.

---

## Erros que aconteceram

### `.git` exposto na internet

O clone estava dentro da pasta pública, e o Caddy serve qualquer arquivo, inclusive os que começam com ponto. Ao ler o log de acesso, apareceu esta linha:

```
uri: /.git/config
User-Agent: ghpv2
status: 200
```

Um robô procurando repositórios expostos pediu `/.git/config` e recebeu 200. O sinal de alerta é o caminho suspeito junto com o `200`. Se fosse 404, estaria protegido.

Correção em três camadas:

1. O clone saiu da pasta pública (`~/projetos`).
2. O deploy copia com `--exclude '.git'`.
3. O Caddy bloqueia arquivos ocultos (guia 09).

Depois do bloqueio, é preciso checar se algo sensível ficou no histórico, porque o robô pode ter baixado antes:

```bash
git remote -v                                         # sem token na URL
git log --all --oneline -- .env '*.env' '*.pem' '*.key'
git ls-files | grep -iE 'env|key|secret|token'
```

Aqui não havia token na URL, nem segredo no histórico, e o repositório é privado.

### Senha pedida no `git pull`

O GitHub não aceita senha de conta no HTTPS. O caminho certo é o token fine-grained.

---

## Manutenção

- **Token:** expira. Anote a data e renove antes. Quando o `git pull` recusar de novo, gere outro.
- **Log:** olhe o `access.log` de vez em quando. Um `200` em `/.git`, `/.env` ou `/wp-admin` é sinal de varredura.
- **Depois de cada deploy:** teste o `200` da página e o `404` do `/.git/config`.

---

## ✅ Resultado esperado

- O site atualiza ao rodar o script, sem o `.git` aparecer
- `/.git/config` retorna 404
- Clone e pasta pública separados

---

## ➡️ Próximo passo

[11 — CrowdSec em modo monitor](./11-crowdsec.md)

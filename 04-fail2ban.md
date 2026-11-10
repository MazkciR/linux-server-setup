# 04 — Fail2ban

## Por que fazer isso?

Mesmo sem login root, os bots continuam tentando outros usuários e senhas. O Fail2ban lê o log do SSH e bane por um tempo o IP que erra a senha várias vezes seguidas.

---

## 1. Instalar

```bash
apt install -y fail2ban
```

---

## 2. Anotar seu IP antes de tudo

Você vai colocar seu IP na lista de exceções. Sem isso, um erro de digitação seu pode te banir do próprio servidor.

No seu PC, descubra o IP público:

```bash
curl ifconfig.me
```

Guarde o resultado. Ele vai entrar no lugar de `SEU_IP_AQUI` no passo 3.

---

## 3. Criar o `jail.local`

Não edite o `jail.conf`: ele é sobrescrito em atualizações. Crie um arquivo só com o que você quer mudar:

```bash
sudo nano /etc/fail2ban/jail.local
```

Cole o conteúdo abaixo:

```ini
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1 SEU_IP_AQUI

[sshd]
enabled  = true
port     = ssh
maxretry = 3
findtime = 10m
bantime  = 1h
logpath  = %(sshd_log)s
backend  = %(sshd_backend)s
```

| Opção | Valor | Significado |
|---|---|---|
| `ignoreip` | localhost + seu IP | IPs que nunca são banidos |
| `maxretry` | 3 | Tentativas erradas antes do ban |
| `findtime` | 10m | Janela de tempo contada |
| `bantime` | 1h | Quanto tempo o IP fica banido |

Salve: `Ctrl+O` → `Enter` → `Ctrl+X`

---

## 4. Ativar

```bash
sudo systemctl enable fail2ban
sudo systemctl restart fail2ban
```

---

## 5. Verificar

```bash
sudo fail2ban-client status sshd
```

Exemplo de saída. Os números variam com o tempo:

```
Status for the jail: sshd
|- Filter
|  |- Currently failed: 2
|  |- Total failed:     24
`- Actions
   |- Currently banned: 4
   `- Banned IP list: 45.x.x.x 172.x.x.x
```

Se `Currently banned` estiver em zero, nada foi banido ainda. Isso é normal em servidor recém-criado, e não quer dizer que o serviço está quebrado.

---

## Como desfazer

Desbanir um IP:

```bash
sudo fail2ban-client set sshd unbanip IP_AQUI
```

Parar o serviço:

```bash
sudo systemctl stop fail2ban
```

---

## Erros que aconteceram

- **`section 'sshd' already exists`:** a seção `[sshd]` apareceu duas vezes no arquivo. Mantenha só uma.
- **Banimento do próprio IP:** errei a senha algumas vezes e fui banido. Resolvi com `unbanip` e, depois, coloquei meu IP no `ignoreip`. Por isso o passo 2 vem antes.

---

## ✅ Resultado esperado

- IPs banidos após 3 tentativas erradas em 10 minutos
- Seu IP e o localhost nunca são banidos

---

## ➡️ Próximo passo

[05 — Chave SSH](./05-chave-ssh.md)

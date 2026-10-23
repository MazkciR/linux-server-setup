# 04 — Fail2ban

## Por que fazer isso?

Mesmo com o root bloqueado, bots continuam tentando invadir seu servidor com outros usuários e senhas. O Fail2ban monitora os logs e bane automaticamente IPs que erram a senha muitas vezes seguidas.

Em menos de 3 minutos após a configuração, já é possível ver IPs sendo banidos.

---

## 1. Instalar

```bash
apt install -y fail2ban
```

---

## 2. Criar arquivo de configuração local

Nunca edite o `jail.conf` diretamente — ele pode ser sobrescrito em atualizações. Use o `.local`:

```bash
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
nano /etc/fail2ban/jail.local
```

---

## 3. Configurar a seção [sshd]

Procure a seção `[sshd]` no arquivo. **Atenção:** ela pode aparecer duas vezes — uma versão incompleta no topo e uma completa no final. Deixe apenas a versão completa com as configurações abaixo:

```ini
[sshd]
enabled = true
port    = ssh
maxretry = 3
findtime = 10m
bantime = 1h
logpath = %(sshd_log)s
backend = %(sshd_backend)s
```

O que cada opção faz:

| Opção | Valor | Significado |
|---|---|---|
| `maxretry` | 3 | Máximo de tentativas erradas |
| `findtime` | 10m | Janela de tempo monitorada |
| `bantime` | 1h | Tempo que o IP fica banido |

---

## 4. Ativar e iniciar

```bash
systemctl enable fail2ban
systemctl restart fail2ban
```

---

## 5. Verificar funcionamento

```bash
fail2ban-client status sshd
```

Saída esperada após alguns minutos:

```
Status for the jail: sshd
|- Filter
|  |- Currently failed: 2
|  |- Total failed:     24
`- Actions
   |- Currently banned: 4
   `- Banned IP list: 45.x.x.x 172.x.x.x ...
```

---

## Problema comum

**Fail2ban não sobe com erro "section 'sshd' already exists"**

Causa: a seção `[sshd]` está duplicada no `jail.local`. Remova a versão incompleta que fica no topo do arquivo e mantenha apenas a versão completa.

---

## ✅ Resultado esperado

- IPs banidos automaticamente após 3 tentativas em 10 minutos
- Proteção ativa contra ataques de força bruta

---

## ➡️ Próximo passo

[05 — Chave SSH](./05-chave-ssh.md)

# 03 — Firewall com UFW

## Por que fazer isso?

Um servidor sem firewall tem todas as portas expostas à internet. O princípio correto é o oposto: **bloquear tudo por padrão e liberar apenas o que você precisa**.

O UFW (Uncomplicated Firewall) é a forma mais simples de fazer isso no Ubuntu.

---

## 1. Instalar o UFW

```bash
apt install -y ufw
```

---

## 2. Liberar SSH antes de ativar

**Importante:** libere a porta SSH antes de ativar o firewall. Se ativar sem liberar, você perde o acesso ao servidor.

```bash
ufw allow ssh
```

---

## 3. Ativar o firewall

```bash
ufw enable
```

Vai perguntar se quer continuar — digite `y`.

---

## 4. Verificar status

```bash
ufw status verbose
```

Saída esperada:

```
Status: active
Default: deny (incoming), allow (outgoing)

To                  Action      From
22/tcp              ALLOW IN    Anywhere
```

---

## Como liberar portas no futuro

Quando instalar aplicações, libere só as portas necessárias:

```bash
# Servidor web
ufw allow 80/tcp   # HTTP
ufw allow 443/tcp  # HTTPS

# Porta específica
ufw allow 3000/tcp

# Bloquear uma porta
ufw deny 3306/tcp
```

---

## ✅ Resultado esperado

- Todo tráfego de entrada bloqueado por padrão
- Apenas porta 22 (SSH) aberta
- Saída de rede liberada normalmente

---

## ➡️ Próximo passo

[04 — Fail2ban](./04-fail2ban.md)

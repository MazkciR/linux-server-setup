# 02 — Hardening SSH

## Por que fazer isso?

O SSH é a porta de entrada do seu servidor. Por padrão, o usuário `root` pode logar via SSH — e bots na internet ficam tentando isso 24 horas por dia. Bloquear o root elimina o alvo mais óbvio.

---

## 1. Editar o arquivo de configuração do SSH

```bash
nano /etc/ssh/sshd_config
```

Procure a linha abaixo e altere para `no`:

```
PermitRootLogin no
```

Salve: `Ctrl+O` → `Enter` → `Ctrl+X`

---

## 2. Verificar overrides

O Ubuntu 24.04 tem uma pasta que pode sobrescrever o `sshd_config`:

```bash
cat /etc/ssh/sshd_config.d/*.conf
```

Se algum arquivo tiver `PasswordAuthentication yes`, altere para `no` agora (você vai precisar disso no passo 05).

---

## 3. Reiniciar o SSH

```bash
systemctl restart ssh
```

---

## ⚠️ Atenção

**Antes de fechar a sessão atual**, abra um novo terminal e teste o login com seu usuário:

```bash
ssh seunome@IP_DA_VPS
```

Se funcionar, pode fechar a sessão root com segurança. Se não testar antes e algo der errado, você pode perder o acesso ao servidor.

---

## ✅ Resultado esperado

Tentativa de login como root bloqueada:

```
Permission denied, please try again.
```

---

## ➡️ Próximo passo

[03 — Firewall com UFW](./03-firewall-ufw.md)

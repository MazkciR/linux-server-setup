# 05 — Autenticação por Chave SSH

## Por que fazer isso?

Senhas podem ser adivinhadas por força bruta. Chaves SSH funcionam de forma diferente: são matematicamente impossíveis de adivinhar. Ao desabilitar o login por senha e usar apenas chaves, você elimina completamente essa classe de ataque.

---

## Como funciona

Uma chave SSH tem duas partes:

- **Chave privada** — fica no seu computador. Nunca compartilhe.
- **Chave pública** — vai para o servidor. Pode ser compartilhada.

O servidor só permite acesso a quem tiver a chave privada correspondente à pública cadastrada.

---

## 1. Gerar a chave no seu computador

No PowerShell (Windows) ou terminal (Linux/Mac):

```bash
ssh-keygen -t ed25519 -C "vps-pessoal"
```

- `-t ed25519` — algoritmo moderno e seguro
- `-C` — comentário para identificar a chave

Quando perguntar onde salvar, pressione **Enter** para o local padrão.
Quando perguntar a passphrase, recomendo definir uma — é uma senha que protege sua chave privada.

---

## 2. Copiar a chave pública para o servidor

**Windows (PowerShell):**

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh seunome@IP_DA_VPS "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

**Linux/Mac:**

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub seunome@IP_DA_VPS
```

---

## 3. Testar antes de continuar

Abra um **novo terminal** e tente logar:

```bash
ssh seunome@IP_DA_VPS
```

Deve pedir a passphrase da chave (não a senha do servidor). Se logar, a chave está funcionando.

---

## 4. Desabilitar login por senha

Edite o arquivo principal:

```bash
nano /etc/ssh/sshd_config
```

```
PasswordAuthentication no
```

Verifique também os overrides:

```bash
nano /etc/ssh/sshd_config.d/50-cloud-init.conf
```

```
PasswordAuthentication no
```

Reinicie o SSH:

```bash
systemctl restart ssh
```

---

## ⚠️ Atenção

- Só desabilite o login por senha **depois** de confirmar que a chave funciona
- Guarde sua chave privada em local seguro — sem ela, você perde o acesso ao servidor
- A passphrase pedida no login é da **chave**, não do servidor — isso é esperado

---

## ✅ Resultado esperado

- Login funciona apenas com chave SSH
- Tentativas de login por senha são recusadas
- Ataques de força bruta por senha tornam-se inúteis

---

## ➡️ Próximo passo

[06 — Unattended Upgrades](./06-unattended-upgrades.md)

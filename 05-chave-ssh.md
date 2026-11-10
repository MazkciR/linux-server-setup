# 05 — Autenticação por Chave SSH

## Por que fazer isso?

Senha pode ser adivinhada por força bruta. Uma chave não: para entrar, é preciso ter a chave privada que corresponde à pública cadastrada no servidor.

---

## Como funciona

- **Chave privada:** fica só no seu computador. Nunca compartilhe.
- **Chave pública:** vai para o servidor. Pode ser compartilhada.

---

## 1. Gerar a chave (no seu PC)

```bash
ssh-keygen -t ed25519 -C "vps-pessoal"
```

- `-t ed25519`: algoritmo moderno e seguro
- `-C`: comentário para identificar a chave

Aperte **Enter** para aceitar o local padrão. Defina uma **passphrase**: é uma senha que protege a chave privada no seu PC.

---

## 2. Copiar a chave pública para a VPS

**Windows (PowerShell):**

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh seunome@IP_DA_VPS "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

**Linux/Mac:**

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub seunome@IP_DA_VPS
```

---

## 3. Ajustar permissões na VPS

Dentro da VPS, rode:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

O SSH é rigoroso com permissões. Se a pasta ou o arquivo estiverem abertos demais, ele ignora a chave em silêncio e volta a pedir senha. É um dos problemas mais comuns nessa etapa.

---

## 4. Testar antes de continuar

Abra um **novo terminal** e entre:

```bash
ssh seunome@IP_DA_VPS
```

Ele deve pedir a **passphrase da chave**, não a senha do servidor. Se entrar, a chave está funcionando.

---

## 5. Desabilitar login por senha

Edite o arquivo principal:

```bash
sudo nano /etc/ssh/sshd_config
```

Procure ou adicione:

```
PasswordAuthentication no
```

Confira se existe algum arquivo extra que sobrescreva essa opção:

```bash
ls /etc/ssh/sshd_config.d/
```

Se aparecer algum `.conf` (por exemplo `50-cloud-init.conf`) com `PasswordAuthentication yes`, troque para `no`. Se não houver nenhum, o passo acima basta.

Reinicie o SSH:

```bash
sudo systemctl restart ssh
```

---

## 6. Confirmar o valor real

O comando abaixo mostra a configuração efetiva, já com todos os arquivos somados:

```bash
sudo sshd -T | grep -i passwordauthentication
```

Saída esperada:

```
passwordauthentication no
```

---

## ⚠️ Antes de desligar a senha

- Só faça o passo 5 depois de confirmar o login com chave (passo 4).
- Guarde a chave privada em local seguro. Sem ela, você fica sem acesso.
- A passphrase é da **chave**, não do servidor. Por isso o terminal a pede.

---

## Se der errado

- **Ele pede a senha do servidor em vez da passphrase:** a chave não está sendo usada. Confira o passo 3 e se a chave pública está em `~/.ssh/authorized_keys`.
- **Ficou trancado fora:** use o console web do painel da hospedagem para entrar e corrigir a configuração. Por isso o passo 4 vem antes do passo 5.

---

## Erros que aconteceram

- **Login por chave não funcionava:** a causa eram permissões abertas demais em `~/.ssh`. Resolvido com o passo 3.

---

## ✅ Resultado esperado

- Login funciona só com a chave
- Tentativa de login por senha é recusada
- `sshd -T` mostra `passwordauthentication no`

---

## ➡️ Próximo passo

[06 — Unattended Upgrades](./06-unattended-upgrades.md)

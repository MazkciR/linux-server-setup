# 01 — Configuração Inicial

## Por que fazer isso?

Servidores recém-criados vêm com pacotes desatualizados. Atualizações de segurança corrigem falhas que invasores exploram ativamente. Fazer isso primeiro garante uma base limpa.

Além disso, trabalhar sempre como `root` é arriscado — qualquer erro ou comando malicioso tem poder total sobre o sistema. Criar um usuário comum com sudo é uma prática básica de segurança.

---

## 1. Atualizar o sistema

```bash
apt update && apt upgrade -y
```

- `apt update` — busca a lista de pacotes disponíveis
- `apt upgrade -y` — instala as atualizações sem pedir confirmação

---

## 2. Criar usuário não-root

```bash
adduser seunome
```

O terminal vai pedir uma senha e alguns dados opcionais (pode apertar Enter para pular).

Adicionar ao grupo sudo:

```bash
usermod -aG sudo seunome
```

---

## 3. Verificar se o sudo funciona

```bash
su - seunome
sudo whoami
```

A saída deve ser `root`. Isso confirma que o usuário tem permissão de executar comandos administrativos quando necessário.

---

## ✅ Resultado esperado

- Sistema atualizado
- Usuário criado e com sudo funcionando
- Pronto para configurar o SSH com segurança

---

## ➡️ Próximo passo

[02 — Hardening SSH](./02-hardening-ssh.md)

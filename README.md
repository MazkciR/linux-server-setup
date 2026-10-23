# 🛡️ Linux Server Setup — Hardening para Iniciantes

> Guia prático para configurar e proteger uma VPS Linux do zero.
> Feito para quem está começando na área de tecnologia e segurança.

---

## 📌 Sobre este repositório

Quando você sobe uma VPS, ela já está sendo atacada — bots varrem a internet em busca de servidores mal configurados 24 horas por dia.

Este guia mostra o mínimo que você precisa fazer antes de colocar qualquer aplicação no ar. Tudo foi aplicado em ambiente real (Ubuntu 24.04 LTS) e documentado passo a passo.

---

## 🖥️ Ambiente

| Item | Detalhe |
|---|---|
| Sistema Operacional | Ubuntu 24.04.4 LTS (Noble Numbat) |
| Acesso | SSH |
| Usuário | Não-root com sudo |

---

## 📋 O que você vai aprender

- Por que não usar o usuário root no dia a dia
- Como configurar um firewall simples e eficiente
- Como bloquear ataques de força bruta automaticamente
- Como eliminar login por senha e usar chaves SSH
- Como monitorar o que acontece no seu servidor

---

## 🗂️ Índice

| Arquivo | Conteúdo |
|---|---|
| [01 — Configuração Inicial](./01-configuracao-inicial.md) | Atualizar sistema e criar usuário seguro |
| [02 — Hardening SSH](./02-hardening-ssh.md) | Bloquear root e configurar acesso seguro |
| [03 — Firewall com UFW](./03-firewall-ufw.md) | Bloquear portas e liberar só o necessário |
| [04 — Fail2ban](./04-fail2ban.md) | Banir IPs com tentativas de invasão |
| [05 — Chave SSH](./05-chave-ssh.md) | Eliminar login por senha completamente |
| [06 — Unattended Upgrades](./06-unattended-upgrades.md) | Atualizações de segurança automáticas |
| [07 — Auditd](./07-auditd.md) | Monitorar acessos e comandos críticos |

---

## ⚡ Checklist rápido

Após subir uma VPS nova, faça isso nesta ordem:

- [ ] Atualizar o sistema
- [ ] Criar usuário não-root com sudo
- [ ] Bloquear login root via SSH
- [ ] Ativar firewall (UFW)
- [ ] Instalar e configurar Fail2ban
- [ ] Configurar autenticação por chave SSH
- [ ] Desabilitar login por senha
- [ ] Verificar Unattended Upgrades
- [ ] Instalar e configurar Auditd

---

## ⚠️ Aviso

Este repositório é para fins educacionais.
Nunca publique IPs, senhas, chaves privadas ou hashes de `/etc/shadow` em repositórios públicos.

---

## 👤 Autor

Feito por [Enrico](https://github.com/Jakeschincariol) como parte do portfólio de estudos em segurança da informação.

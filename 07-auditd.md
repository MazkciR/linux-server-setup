# 07 — Auditd

## Por que fazer isso?

Se algo suspeito acontecer no seu servidor, você precisa saber o que foi acessado, quando e por quem. O Auditd registra eventos críticos do sistema: arquivos acessados, comandos executados como root, tentativas de acesso negado.

É a diferença entre investigar um incidente com evidências ou no escuro.

---

## 1. Instalar

```bash
apt install -y auditd
systemctl status auditd
```

---

## 2. Adicionar regras de monitoramento

```bash
nano /etc/audit/rules.d/audit.rules
```

Cole as regras abaixo:

```bash
# Monitora alterações em arquivos críticos do sistema
-w /etc/passwd -p wa -k usuarios
-w /etc/shadow -p wa -k senhas
-w /etc/ssh/sshd_config -p wa -k ssh_config
-w /etc/sudoers -p wa -k sudo

# Registra comandos executados como root
-a always,exit -F arch=b64 -S execve -F euid=0 -F key=root_commands

# Registra tentativas de acesso negado
-a always,exit -F arch=b64 -S open -F exit=-EACCES -F key=acesso_negado
```

O que cada parte significa:

| Opção | Significado |
|---|---|
| `-w /caminho` | Monitora esse arquivo |
| `-p wa` | Detecta escrita (`w`) e mudança de atributo (`a`) |
| `-k nome` | Tag para facilitar a busca nos logs |
| `-F euid=0` | Filtra apenas o usuário root (UID 0) |

---

## 3. Carregar as regras

```bash
augenrules --load
auditctl -l
```

A saída deve listar todas as regras adicionadas.

---

## 4. Como consultar os logs

```bash
# Ver acessos ao /etc/shadow
sudo ausearch -k senhas

# Ver comandos executados como root recentemente
sudo ausearch -k root_commands --start recent

# Ver acessos negados
sudo ausearch -k acesso_negado --start recent
```

---

## ✅ Resultado esperado

- Qualquer alteração em arquivos críticos é registrada
- Comandos executados como root ficam auditáveis
- Tentativas de acesso não autorizado são logadas

---

## 🏁 Hardening concluído

Seu servidor agora tem uma base sólida de segurança:

| Camada | Ferramenta |
|---|---|
| Firewall | UFW |
| Proteção contra força bruta | Fail2ban |
| Acesso seguro | Chave SSH Ed25519 |
| Atualizações automáticas | Unattended Upgrades |
| Monitoramento | Auditd |

---

## Referências

- [CIS Benchmarks Ubuntu](https://www.cisecurity.org/benchmark/ubuntu_linux)
- [Fail2ban docs](https://github.com/fail2ban/fail2ban)
- [UFW manual](https://manpages.ubuntu.com/manpages/noble/en/man8/ufw.8.html)
- [Linux Audit Documentation](https://github.com/linux-audit/audit-documentation)

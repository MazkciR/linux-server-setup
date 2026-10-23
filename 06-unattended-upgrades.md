# 06 — Unattended Upgrades

## Por que fazer isso?

Vulnerabilidades novas são descobertas o tempo todo. Quando uma falha de segurança é corrigida, a atualização precisa chegar ao seu servidor o quanto antes. O Unattended Upgrades faz isso automaticamente, sem precisar lembrar de rodar `apt upgrade` manualmente.

---

## Verificar se está ativo

No Ubuntu 24.04, ele já vem instalado e ativo por padrão. Verifique:

```bash
systemctl status unattended-upgrades
```

Saída esperada: `active (running)`

Confirme a configuração:

```bash
cat /etc/apt/apt.conf.d/20auto-upgrades
```

Saída esperada:

```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

- `Update-Package-Lists "1"` — busca atualizações diariamente
- `Unattended-Upgrade "1"` — instala automaticamente as de segurança

---

## Se não estiver instalado

```bash
apt install -y unattended-upgrades
dpkg-reconfigure --priority=low unattended-upgrades
```

---

## ✅ Resultado esperado

- Atualizações de segurança aplicadas automaticamente
- Sem necessidade de intervenção manual para patches críticos

---

## ➡️ Próximo passo

[07 — Auditd](./07-auditd.md)

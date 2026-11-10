# 11 — CrowdSec em modo monitor

## Por que usar?

O Fail2ban olha o SSH. O CrowdSec lê vários logs ao mesmo tempo (SSH, auditd, Caddy) e reconhece padrões de ataque, como varredura de rotas e força bruta. Ele também baixa uma lista coletiva de IPs maliciosos, mantida pela comunidade.

Aqui ele roda em **modo monitor**: detecta e registra, mas **não bloqueia nada**. Isso evita que um falso positivo te tire do servidor.

---

## 1. Instalar

Siga a documentação oficial: https://docs.crowdsec.net/

Depois confira:

```bash
sudo systemctl status crowdsec
sudo cscli metrics
```

---

## 2. Desativar o bouncer

O bouncer é o componente que aplica os bloqueios no firewall. Para só observar, desligue-o:

```bash
sudo systemctl disable --now crowdsec-firewall-bouncer
```

---

## 3. Fontes de log

O CrowdSec lê o que estiver em `/etc/crowdsec/acquis.d/`. Para incluir o log de acesso do Caddy, crie `caddy.yaml`:

```yaml
filenames:
  - /home/SEU_USUARIO/caddy/logs/access.log
labels:
  type: caddy
```

Reinicie e confirme que ele está lendo:

```bash
sudo systemctl restart crowdsec
sudo cscli metrics
```

O arquivo de log precisa estar legível. Se ele nasceu com permissão 600, o CrowdSec não consegue abrir (veja o guia 09, `mode 0644`).

---

## 4. Métricas para o Prometheus

No `/etc/crowdsec/config.yaml`, a seção `prometheus` deve ficar assim:

```yaml
prometheus:
  enabled: true
  level: full
  listen_addr: 127.0.0.1
  listen_port: 6060
```

Teste:

```bash
curl -s http://127.0.0.1:6060/metrics | grep -E '^cs_' | cut -d'{' -f1 | sort -u | head
```

A porta fica só no loopback. O guia 12 mostra como o Prometheus coleta.

---

## 5. Comandos do dia a dia

```bash
sudo cscli alerts list              # detecções locais
sudo cscli decisions list           # IPs com decisão ativa
sudo cscli decisions list | grep -v CAPI   # só as decisões locais
sudo cscli capi status              # conexão com a lista coletiva
```

---

## Decisão local x lista coletiva (CAPI)

Ao ver `cscli decisions list`, a coluna de origem mostra de onde veio cada decisão:

- **CAPI:** lista coletiva da comunidade. Pode ter milhares de IPs (aqui eram cerca de 15 mil). Não são ataques que o seu servidor sofreu.
- **crowdsec / cscli:** decisões geradas pelo seu próprio servidor ou criadas por você.

Os números do painel de segurança só fazem sentido se você separar essas duas origens.

---

## Erros que aconteceram

- **Bloqueado do próprio servidor:** com o bouncer ativo, o CrowdSec bloqueou a porta 22. Diagnóstico: `sudo cscli decisions list` e `sudo iptables -L INPUT -n --line-numbers`. Solução: `sudo cscli decisions delete --ip SEU_IP`, e desativar o bouncer.
- **Métrica mostrando bloqueio que já expirou:** o `cs_active_decisions` ficou com um valor antigo. O `cscli` dizia que não havia decisão ativa. Reiniciar o serviço (`sudo systemctl restart crowdsec`) atualiza.
- **Timestamp da lista coletiva zerado após reiniciar:** normal. O CrowdSec ainda não fez a primeira atualização. Confirme a conexão com `sudo cscli capi status`.

---

## ✅ Resultado esperado

- `cscli metrics` mostra as fontes de log sendo lidas
- `cscli capi status` mostra conexão ok com a lista coletiva
- Nada está sendo bloqueado: só observado

---

## ➡️ Próximo passo

[12 — Monitoramento com Prometheus e Grafana](./12-monitoramento.md)

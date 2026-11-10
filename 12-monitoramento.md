# 12 — Monitoramento: Prometheus, Grafana e Netdata

## Por que monitorar?

Hardening sem monitoramento é fé. Os painéis mostram se o servidor está saudável, se alguém está atacando e se as proteções estão funcionando.

Tudo roda **local**: nada é exposto na internet. O acesso é por túnel SSH.

---

## O que faz o quê

| Peça | Função | Endereço |
|---|---|---|
| **Netdata** | Coleta CPU, RAM, disco e rede do servidor | `127.0.0.1:19999` |
| **CrowdSec** | Expõe métricas de ataques e bloqueios | `127.0.0.1:6060` |
| **Prometheus** | Guarda as métricas ao longo do tempo | `127.0.0.1:9090` |
| **Grafana** | Mostra tudo em painéis | `127.0.0.1:3000` |

Instale o Netdata conforme a documentação oficial e confirme que ele escuta só no loopback com `sudo ss -tlnp | grep 19999`.

---

## 1. docker-compose.yml

Em `~/monitoramento/docker-compose.yml` (versão simplificada; em uso real, fixe as versões das imagens):

```yaml
services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    restart: unless-stopped
    network_mode: host
    command:
      - --config.file=/etc/prometheus/prometheus.yml
      - --web.listen-address=127.0.0.1:9090
      - --storage.tsdb.retention.time=30d
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus-data:/prometheus

  grafana:
    image: grafana/grafana
    container_name: grafana
    restart: unless-stopped
    network_mode: host
    environment:
      - GF_SERVER_HTTP_ADDR=127.0.0.1
      - GF_SERVER_HTTP_PORT=3000
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASS}
    volumes:
      - grafana-data:/var/lib/grafana

volumes:
  prometheus-data:
  grafana-data:
```

Por que `network_mode: host`: o Netdata e o CrowdSec escutam em `127.0.0.1` do servidor. Um container em rede comum não chega lá. Em modo host, o container usa a rede do servidor e enxerga esses endereços. Como cada serviço escuta só em `127.0.0.1`, nada fica exposto.

A senha do Grafana vem do arquivo `.env` (modo 600), na variável `GRAFANA_PASS`. Nunca coloque o valor no compose.

---

## 2. prometheus.yml

```yaml
global:
  scrape_interval: 30s

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['127.0.0.1:9090']

  - job_name: crowdsec
    static_configs:
      - targets: ['127.0.0.1:6060']

  - job_name: netdata
    metrics_path: /api/v1/allmetrics
    params:
      format: ['prometheus']
    static_configs:
      - targets: ['127.0.0.1:19999']
```

Suba tudo:

```bash
cd ~/monitoramento
docker compose up -d
```

Confira se os três alvos estão ativos (valor `"1"`):

```bash
curl -s 'http://127.0.0.1:9090/api/v1/query?query=up' | python3 -m json.tool | grep -E '"job"|"value"'
```

---

## 3. Acessar pelo seu PC (túnel SSH)

No PC, edite `~/.ssh/config` (no Windows: `C:\Users\SEU_USUARIO\.ssh\config`):

```
Host vps
    HostName IP_DA_VPS
    User SEU_USUARIO
    LocalForward 3000 127.0.0.1:3000
    LocalForward 9090 127.0.0.1:9090
    LocalForward 19999 127.0.0.1:19999
```

Abra o túnel e deixe a janela aberta:

```bash
ssh -N vps
```

No navegador:

- Grafana: `http://localhost:3000` (usuário `admin`)
- Prometheus: `http://localhost:9090`
- Netdata: `http://localhost:19999`

---

## 4. Ligar o Grafana ao Prometheus

No Grafana: **Connections → Data sources → Add → Prometheus**, URL `http://127.0.0.1:9090`. Salve e teste.

---

## 5. Consultas úteis

Use no **Explore** ou nos painéis:

| O que mostra | Consulta |
|---|---|
| Alvos no ar | `up` |
| CPU em uso (%) | `100 - netdata_system_cpu_percentage_average{dimension="idle"}` |
| RAM usada (%) | `100 * netdata_system_ram_MiB_average{dimension="used"} / scalar(sum(netdata_system_ram_MiB_average))` |
| Disco `/` usado (%) | `100 * netdata_disk_space_GiB_average{family="/",dimension="used"} / scalar(sum(netdata_disk_space_GiB_average{family="/"}))` |
| Alertas do CrowdSec no período | `sum(increase(cs_alerts[$__range]))` |
| Bloqueios locais ativos | `sum(cs_active_decisions{origin!="CAPI"}) or vector(0)` |
| Alertas por tipo | `sum by (reason) (increase(cs_alerts[15m]))` |

Antes de montar um painel, descubra os nomes reais das métricas:

```bash
curl -s 'http://127.0.0.1:19999/api/v1/allmetrics?format=prometheus' | grep -E '^netdata_(system_cpu|system_ram|disk_space)' | cut -d'{' -f1 | sort -u
```

---

## Erros que aconteceram

- **Dashboard importado vazio (`A data source is required`):** o ID que escolhi era de outro projeto, não de servidor. Antes de importar, confira o que o dashboard mede. Ou monte o seu.
- **"No data" em indicador de porcentagem:** dividir uma série com rótulos por um `sum()` sem rótulos não casa no Prometheus. A solução é `scalar(sum(...))`.
- **Indicador preso em valor antigo:** quando uma série some, o Grafana mostra o último valor que existiu. Para mostrar 0, use `... or vector(0)`.
- **"2 de 3 alvos no ar":** o Prometheus não coletava a si mesmo. Faltava o job `prometheus`.
- **Comandos do cmd colados no PowerShell:** parênteses e `%USERPROFILE%` não funcionam no PowerShell. Use `$env:USERPROFILE` ou abra um `cmd`. Ao criar o `config` do SSH, o PowerShell 5.1 pode gravar com BOM, o que quebra o arquivo.

---

## ✅ Resultado esperado

- `up` mostra os três alvos com valor 1
- Grafana abre em `http://localhost:3000` pelo túnel
- Você vê CPU, RAM, disco e os alertas do CrowdSec num painel

---

## ➡️ Próximo passo

[13 — n8n com Postgres e redes Docker](./13-n8n.md)

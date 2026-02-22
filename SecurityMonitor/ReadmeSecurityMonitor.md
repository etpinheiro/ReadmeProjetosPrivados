<h1 align="center">
  <img src="https://img.shields.io/badge/📡-Security%20Monitor-10b981?style=for-the-badge&labelColor=0a0e17" alt="Security Monitor"/>
</h1>

<h3 align="center">Sistema de Monitoramento de Equipamentos em Tempo Real</h3>

<p align="center">
  <img src="https://img.shields.io/badge/Versão-2.1%20Multi--Área-10b981?style=flat-square"/>
  <img src="https://img.shields.io/badge/Ping-24%2F7%20a%20cada%205s-3b82f6?style=flat-square"/>
  <img src="https://img.shields.io/badge/Meta%20Uptime-95%25-eab308?style=flat-square"/>
  <img src="https://img.shields.io/badge/Licença-Proprietário-ef4444?style=flat-square"/>
</p>

<p align="center">
  Plataforma web para monitoramento 24/7 de câmeras CFTV e controles de acesso distribuídos em múltiplas áreas geográficas.<br>
  Pings automáticos a cada 5 segundos com 50 threads paralelas, dashboards em tempo real e exportação para Power BI.
</p>

---

### Sobre o Projeto

O **Security Monitor v2.1** foi desenvolvido para resolver o problema de monitoramento descentralizado de equipamentos de segurança patrimonial em ambientes com múltiplas áreas geográficas. O sistema detecta automaticamente falhas, calcula uptime/downtime com meta de 95%, gerencia workflows de manutenção e exporta dados consolidados para Power BI.

**Problemas que resolve:**
- Equipamentos offline sem detecção automática — falhas passavam despercebidas por horas
- Sem cálculo centralizado de uptime/downtime por área, tipo e contexto
- Manutenções sem workflow formal (relatório inicial → reparo → relatório final)
- Dados de monitoramento isolados, sem integração com ferramentas de BI
- Áreas remotas com alta latência de ping — necessidade de agentes locais

---

### Funcionalidades

**Monitoramento em Tempo Real**
- Pings automáticos a cada 5 segundos usando ThreadPoolExecutor com 50 workers paralelos
- Detecção automática de falhas (ONLINE → OFFLINE) com registro de log
- Captura de snapshot de câmeras via HTTP Digest/Basic
- 4 estados de equipamento: ONLINE (verde), OFFLINE (vermelho), MANUTENÇÃO (amarelo), DESLIGADO (cinza)
- Equipamentos em manutenção/desligados são excluídos do ciclo de ping automaticamente

**Sistema Multi-Área com Agentes Remotos**
- Suporte a múltiplas áreas geográficas com código único (ex: ITA, SNT, PAR)
- Agentes remotos independentes pingam equipamentos localmente com menor latência
- Heartbeat + sincronização via API REST com autenticação por API key
- Buffer SQLite local no agente para quando o servidor central está offline
- Fallback automático: se agente cai (>180s sem heartbeat), o servidor assume o ping

**Dashboard e Métricas**
- Cards de status com contadores agrupados por tipo/contexto
- Gráficos interativos com Chart.js 4.4.1 (uptime histórico, timeline de quedas)
- Auto-refresh a cada 30 segundos
- Cálculo de uptime/downtime mensal e total com meta de 95%
- Histórico detalhado por equipamento com gráficos

**Workflow de Manutenção**
- Relatório inicial: operador descreve o problema → equipamento entra em MANUTENÇÃO
- Relatório final: técnico descreve a solução → equipamento volta a ser pingado
- 7 categorias de problema: Rede, Hardware, Software, Energia, Configuração, Infraestrutura, Outro
- Solicitações de desligamento com aprovação por admin

**Exportação para Power BI**
- BIExportService gera snapshots diários e mensais automaticamente
- Exportação CSV por área (uma pasta por código de área)
- Rotina a cada 20 minutos + rotina de meia-noite para virada de mês
- Cálculo automático de meta 95%, percentual faltante e uptime anual
- Reset de contadores mensais na virada do mês sem perda de dados

**Segurança e Permissões**
- Senhas com hash scrypt via Werkzeug (mínimo 12 chars, maiúscula, minúscula, dígito, especial)
- Permissões granulares por tela (9 telas × 2 ações: pode_ver / pode_editar)
- Decorators `@permissao_requerida` e `@admin_requerido`
- Headers HTTP de segurança (X-Frame-Options, X-XSS-Protection, CSP, HSTS)
- Filtro por área via cookie — operador só vê equipamentos da sua área

---

### Arquitetura

```
+---------------------------------------------------------------+
|                      NAVEGADOR (Browser)                      |
|  Dashboard | Câmeras | Relatórios | Configuração | Histórico  |
+-----------------------------+---------------------------------+
                              | API REST (fetch)
+-----------------------------+---------------------------------+
|           SERVIDOR FLASK (run.py - porta 5000)                |
|  +---------------------------------------------------------+  |
|  | Blueprints: main_bp, auth_bp, api_bp, api_agente        |  |
|  +---------------------------------------------------------+  |
|  | PingService (50 threads)  | BIExportService (thread)    |  |
|  +------+--------+----------+--------+--------------------+  |
|         |        |                   |                        |
|  +------+--------+   +--------+-----+----+                   |
|  | SQLAlchemy ORM |   | CSV Export (BI)   |                   |
|  | QueuePool 20+40|   | exports/{AREA}/   |                   |
|  +------+---------+   +------------------+                   |
+---------+------------------------------------------------+---+
          |
+---------+------------------------------------------------+
|        SQL Server Express (SecurityMonitor)               |
|  Windows Authentication | ODBC Driver 17 | QueuePool      |
+-----------------------------------------------------------+

  +------------------+     +------------------+
  | AGENTE REMOTO    | --> | /api/agente/*    | (heartbeat + sync)
  | (Área local)     |     | (servidor)       |
  | SQLite buffer    |     +------------------+
  +------------------+
```

---

### Tipos de Equipamento

| Tipo | Contexto | Descrição |
|---|---|---|
| Câmera | RFB | Câmeras de CFTV (contexto governamental) |
| Câmera | Normais | Câmeras de CFTV convencionais |
| Controle | RFB | Controles de acesso (contexto governamental) |
| Controle | Normais | Controles de acesso convencionais |

### Estados de Equipamento

| Status | Cor | Pingado? | Conta como downtime? |
|---|---|---|---|
| **ONLINE** | 🟢 Verde | Sim | Não (é uptime) |
| **OFFLINE** | 🔴 Vermelho | Sim | Sim |
| **MANUTENÇÃO** | 🟡 Amarelo | Não | Não |
| **DESLIGADO** | ⚪ Cinza | Não | Não |

---

### Tech Stack

**Backend**

![Python](https://img.shields.io/badge/Python_3.x-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask_3.0-000000?style=for-the-badge&logo=flask&logoColor=white)
![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy_2.0-D71F00?style=for-the-badge&logo=sqlalchemy&logoColor=white)
![APScheduler](https://img.shields.io/badge/APScheduler-4B8BBE?style=for-the-badge&logo=clockify&logoColor=white)

**Frontend**

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Chart.js](https://img.shields.io/badge/Chart.js_4.4-FF6384?style=for-the-badge&logo=chartdotjs&logoColor=white)

**Banco de Dados & Infra**

![SQL Server](https://img.shields.io/badge/SQL%20Server%20Express-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)
![pyodbc](https://img.shields.io/badge/pyodbc-003B57?style=for-the-badge&logo=odbc&logoColor=white)
![Windows](https://img.shields.io/badge/Windows%20Server-0078D4?style=for-the-badge&logo=windows&logoColor=white)

**Monitoramento & BI**

![ICMP Ping](https://img.shields.io/badge/ICMP%20Ping-009688?style=for-the-badge&logo=speedtest&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI%20(CSV)-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Threading](https://img.shields.io/badge/50%20Threads-6366f1?style=for-the-badge&logo=turborepo&logoColor=white)

---

### Banco de Dados — 12 Tabelas

| Tabela | Descrição | Campos Principais |
|---|---|---|
| `usuarios` | Usuários do sistema | username, password_hash (scrypt), is_admin, ativo |
| `permissoes` | Permissões por tela | usuario_id, tela, pode_ver, pode_editar |
| `equipamentos` | Câmeras e controles (25+ colunas) | tipo, ip, status_atual, uptime_total, downtime_total, area_id |
| `areas` | Áreas geográficas | codigo (UNIQUE), agente_url, agente_api_key, agente_status |
| `agentes_heartbeat` | Heartbeat dos agentes remotos | area_id, status, latencia_ms, equipamentos_online |
| `status_log` | Histórico de mudanças de status | equipamento_id, status_anterior, status_novo, timestamp |
| `ping_log` | Log individual de cada ping | equipamento_id, online, latencia_ms, timestamp |
| `relatorios` | Relatórios de manutenção | equipamento_id, tipo (inicial/final), descricao, categoria |
| `solicit_desligamento` | Workflow de desligamento | equipamento_id, justificativa, status, aprovado_por |
| `config_sistema` | Configurações key-value | chave (UNIQUE), valor |
| `snapshots_diarios` | Snapshot diário para BI | equipamento_id, data, uptime_segundos, downtime_segundos |
| `snapshots_mensais` | Snapshot mensal para BI | equipamento_id, mes_ano, uptime_percent, meta_atingida |

---

### API REST (~2.900 linhas)

Padrão de resposta: `{"success": bool, "data": {...}, "error": "msg", "area_id": 1}`

| Recurso | Endpoints | Operações |
|---|---|---|
| Equipamentos | `GET/POST/PUT/DELETE /api/equipamentos` | CRUD + distribuição, pendentes, manutenção, desligados |
| Ping | `POST /api/equipamentos/<id>/ping` | Ping manual + atualização de status |
| Dashboard | `GET /api/dashboard/stats` | Contadores, histórico uptime (30 dias), últimas mudanças |
| Relatórios | `GET/POST /api/relatorios` | Criar relatório inicial/final, listar com filtros |
| Solicitações | `GET/POST /api/solicitacoes` | Workflow desligamento: criar, aprovar, rejeitar |
| Usuários | `GET/POST/PUT/DELETE /api/usuarios` | CRUD com validação de senha forte |
| Configurações | `GET/PUT /api/configuracoes` | Configurações do sistema |
| Snapshot | `GET /api/equipamentos/<id>/snapshot` | Captura imagem da câmera (HTTP Digest) |
| Agente | `POST /api/agente/heartbeat` | Heartbeat + ping-results + sync batch |

**API do Agente Remoto (~1.500 linhas):** 8 endpoints dedicados para comunicação com agentes, incluindo heartbeat, sincronização de ping em lote e envio de snapshots diários/mensais.

---

### Serviço de Ping (PingService)

| Parâmetro | Valor | Descrição |
|---|---|---|
| Intervalo | **5 segundos** | Ciclo entre cada rodada de pings |
| Timeout | **2 segundos** | Timeout individual por ping |
| Workers | **50 threads** | ThreadPoolExecutor paralelo |
| Fallback agente | **180 segundos** | Tempo sem heartbeat para servidor assumir |
| Cap duração | **300 segundos** | Máximo de tempo contado por ciclo |

**Fluxo do ciclo:**
1. Verificar áreas com agentes ativos (não pinga se agente OK)
2. Buscar equipamentos ativos (excluir MANUTENÇÃO/DESLIGADO)
3. Executar pings em paralelo (ping3 → fallback subprocess)
4. Incrementar contadores uptime/downtime por equipamento
5. Registrar mudanças de status em `status_log`
6. Commit + sleep(5) → repetir

---

### Estrutura do Projeto

```
SecurityMonitor/
├── run.py                          # Entry point (argparse: host, port, debug)
├── SecurityMonitor.pyw             # Launcher Windows (sem console)
├── INICIAR_SERVIDOR.bat            # Script de inicialização Windows
├── requirements.txt                # Dependências Python
├── .env                            # Variáveis de ambiente (NÃO versionado)
├── config/
│   └── settings.py                 # Classes Config (Dev/Prod/Test)
├── app/
│   ├── __init__.py                 # Application Factory + Blueprints
│   ├── database.py                 # Engine SQLAlchemy + DatabaseSession
│   ├── decorators.py               # @permissao_requerida, @admin_requerido
│   ├── models/                     # 10 models ORM (SQLAlchemy 2.0)
│   │   ├── usuario.py              # Autenticação com scrypt
│   │   ├── equipamento.py          # Equipamentos (25+ colunas)
│   │   ├── area.py                 # Áreas + AgenteHeartbeat
│   │   ├── permissao.py            # Permissões por tela
│   │   ├── status_log.py           # Log de mudanças de status
│   │   ├── ping_log.py             # Log de pings individuais
│   │   ├── relatorio.py            # Relatórios de manutenção
│   │   ├── solicitacao_deslig.py   # Solicitações de desligamento
│   │   ├── configuracao_sistema.py # Configurações key-value
│   │   └── snapshot_bi.py          # Snapshots diários/mensais (BI)
│   ├── routes/                     # 4 Blueprints
│   │   ├── main.py                 # Rotas de páginas (233 linhas)
│   │   ├── auth.py                 # Login, logout, alterar senha
│   │   ├── api.py                  # API REST principal (~2.900 linhas)
│   │   └── api_agente.py           # API agentes remotos (~1.500 linhas)
│   ├── services/                   # Serviços de background
│   │   ├── ping_service.py         # Serviço de ping automático (332 linhas)
│   │   └── bi_export_service.py    # Exportação CSV para Power BI (787 linhas)
│   ├── templates/                  # 9 templates HTML (Jinja2)
│   └── static/
│       ├── css/style.css           # Estilos (~3.500 linhas) com dark/light mode
│       └── js/app.js               # JavaScript principal (589 linhas)
├── database/
│   └── 01_CRIAR_BANCO_E_TABELAS.sql  # Schema completo (~400 linhas)
└── exports/                        # CSVs gerados para Power BI
    └── {AREA_CODE}/                # Uma pasta por área
        ├── snapshots_diarios.csv
        └── snapshots_mensais.csv
```

---

### Fluxos Operacionais

**Fluxo de Manutenção:**
```
Equipamento OFFLINE → Operador cria Relatório Inicial → Status: MANUTENÇÃO (para de pingar)
→ Técnico repara → Relatório Final → Status: OFFLINE (volta a pingar) → Ping detecta → ONLINE
```

**Fluxo de Desligamento:**
```
Operador solicita desligamento → Admin aprova → Status: DESLIGADO (para de pingar)
→ Religamento → Status: OFFLINE → Ping detecta → ONLINE
```

**Fluxo do Agente Remoto:**
```
Agente busca config → Pinga equipamentos locais → Envia heartbeat + resultados
→ Se servidor offline: buffer SQLite → Quando volta: sync batch
→ Se agente cai (>180s): servidor ativa fallback automático
```

---

### Dependências Principais

```
Flask>=3.0.0
Flask-Login>=0.6.3
Flask-WTF>=1.2.0
pyodbc>=5.0.0
SQLAlchemy>=2.0.0
Werkzeug>=3.0.0
APScheduler>=3.10.0
ping3>=4.0.0
Chart.js 4.4.1 (frontend)
requests>=2.31.0
python-dotenv>=1.0.0
```

---

### Autor

Desenvolvido por **Erik Thiago Barros Pinheiro**

<p align="center">
  <a href="https://wa.me/5591992994172"><img src="https://img.shields.io/badge/WhatsApp-25D366?style=for-the-badge&logo=whatsapp&logoColor=white"/></a>
  <a href="mailto:erikthiagobarrospinheiro349@gmail.com"><img src="https://img.shields.io/badge/Gmail-EA4335?style=for-the-badge&logo=gmail&logoColor=white"/></a>
  <a href="https://instagram.com/erikdevelopments"><img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white"/></a>
  <a href="https://github.com/etpinheiro"><img src="https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white"/></a>
</p>

---

```
Copyright (c) 2026 Erik Thiago Barros Pinheiro. Todos os direitos reservados.
Código-fonte proprietário e confidencial. Nomes de empresas omitidos por NDA.
```

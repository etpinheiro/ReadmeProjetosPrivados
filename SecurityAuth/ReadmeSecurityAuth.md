<h1 align="center">
  <img src="https://img.shields.io/badge/🔐-SecurityAuth-00d4ff?style=for-the-badge&labelColor=0a0e17" alt="SecurityAuth"/>
</h1>

<h3 align="center">Sistema de Autenticação TACACS+ para Dispositivos de Rede</h3>

<p align="center">
  <img src="https://img.shields.io/badge/Versão-1.0-00d4ff?style=flat-square"/>
  <img src="https://img.shields.io/badge/Protocolo-RFC%208907-10b981?style=flat-square"/>
  <img src="https://img.shields.io/badge/Status-Em%20Produção-10b981?style=flat-square"/>
  <img src="https://img.shields.io/badge/Licença-Proprietário-ef4444?style=flat-square"/>
</p>

<p align="center">
  Sistema completo de gerenciamento de autenticação de rede baseado no protocolo TACACS+ (RFC 8907).<br>
  Centraliza o controle de acesso a switches e roteadores Cisco, eliminando senhas locais por dispositivo.
</p>

---

### Sobre o Projeto

O **SecurityAuth** foi desenvolvido para resolver o problema de autenticação descentralizada em ambientes com múltiplos dispositivos de rede. Em vez de gerenciar senhas locais em cada switch/roteador Cisco individualmente, o sistema centraliza toda a autenticação, autorização e contabilização (AAA) em um único servidor TACACS+ com painel web.

**Problema que resolve:**
- Senhas locais espalhadas em dezenas de switches sem controle centralizado
- Sem auditoria de quem acessou qual dispositivo e executou quais comandos
- Sem controle granular de permissões por grupo/nível de privilégio
- Necessidade de acesso SSH remoto aos switches sem cliente instalado

---

### Funcionalidades

**Servidor TACACS+ Próprio (porta TCP 49)**
- Implementação completa do protocolo conforme RFC 8907
- Autenticação ASCII e PAP com criptografia MD5 do corpo inteiro do pacote
- Autorização por políticas: comandos permitidos/negados via regex por grupo
- Contabilização (accounting): registro de início/fim de sessão e comandos executados
- Cada conexão tratada em thread separada com timeout de 30 segundos

**Painel Web Completo**
- Dashboard com estatísticas em tempo real (auto-refresh 30s)
- CRUD completo de switches, usuários, grupos e políticas de autorização
- Sistema de permissões granulares por grupo (7 páginas × 6 ações)
- 3 perfis pré-definidos: Admin, Suporte e Visualizador (customizáveis)
- Logs de acesso com filtros, paginação e exportação
- Tema escuro customizado com CSS variables

**Terminal SSH via Navegador**
- Acesso direto aos switches pelo browser usando WebSocket + xterm.js
- Tema Catppuccin Mocha com 5000 linhas de scrollback
- Sidebar com switches agrupados, busca e drag-drop entre grupos
- Modal automático de credenciais quando o switch exige login
- Suporte a redimensionamento de terminal em tempo real

**Segurança**
- Rate limiting: 5 tentativas em 5 min, bloqueio de 15 min por IP
- Fingerprint de sessão via SHA-256 (User-Agent + Accept-Language)
- Timeout por inatividade de 60 minutos
- Headers HTTP de segurança (HSTS, X-Frame-Options, CSP, etc.)
- Proteção CSRF com Flask-WTF (token de 1 hora)
- Cookies HttpOnly + SameSite=Lax (Secure em produção)
- Senhas com hash via Werkzeug (scrypt)
- Sanitização de input com validação regex

**Migração do TACACS.NET**
- Script para importar dados de instalações TACACS.NET existentes
- Lê arquivos XML de configuração (clients.xml + authentication.xml)
- Cria switches, grupos e usuários automaticamente

---

### Arquitetura

```
+-------------------+     +-------------------+     +------------------+
| Switches Cisco    |     | Navegador Web     |     | CMD/SSH Client   |
| (TACACS+ client)  |     | (Dashboard/Term)  |     |                  |
+--------+----------+     +--------+----------+     +--------+---------+
         |TCP:49                   |HTTP/WS:5002              |SSH:22
         v                         v                          v
+--------+-------------------------+-------------------------+---------+
|                     SERVIDOR SECURITYAUTH                            |
|  +----------------+  +----------------+  +------------------------+  |
|  | TACACS+ Server |  | Flask Web App  |  | SSH Proxy (Paramiko)   |  |
|  | (porta 49)     |  | (porta 5002)   |  | (via WebSocket)        |  |
|  +-------+--------+  +-------+--------+  +----------+-------------+  |
|          |                    |                       |               |
|          +--------------------+-----------------------+              |
|                               |                                      |
|                    +----------+----------+                           |
|                    | SQL Server / SQLite |                           |
|                    +---------------------+                           |
+----------------------------------------------------------------------+
```

**Fluxo de Autenticação TACACS+:**

```
Switch                  Servidor SecurityAuth        Banco de Dados
  |                            |                           |
  |--- TCP Connect (porta 49) -->|                         |
  |--- AUTH START (user) ------->|                         |
  |<-- GETPASS (solicita senha) -|                         |
  |--- AUTH CONTINUE (senha) --->|                         |
  |                              |--- SELECT user -------->|
  |                              |<-- password_hash -------|
  |                              |--- check_password() --->|
  |<-- AUTH PASS/FAIL -----------|--- INSERT access_log -->|
  |                              |                         |
  |--- AUTHOR REQUEST (cmd) ---->|                         |
  |                              |--- SELECT policies ---->|
  |<-- AUTHOR PASS/FAIL --------|                         |
  |                              |                         |
  |--- ACCT START/STOP -------->|--- INSERT access_log -->|
  |<-- ACCT SUCCESS ------------|                         |
```

---

### Tech Stack

**Backend**

![Python](https://img.shields.io/badge/Python_3.x-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask_3.0-000000?style=for-the-badge&logo=flask&logoColor=white)
![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy_3.1-D71F00?style=for-the-badge&logo=sqlalchemy&logoColor=white)
![Gunicorn](https://img.shields.io/badge/Gunicorn-499848?style=for-the-badge&logo=gunicorn&logoColor=white)
![Paramiko](https://img.shields.io/badge/Paramiko_3.4-2b2b2b?style=for-the-badge&logo=gnometerminal&logoColor=white)

**Frontend**

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Tailwind](https://img.shields.io/badge/Tailwind_CSS-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white)
![xterm.js](https://img.shields.io/badge/xterm.js_5.3-000000?style=for-the-badge&logo=windowsterminal&logoColor=white)

**Banco de Dados & Infra**

![SQL Server](https://img.shields.io/badge/SQL%20Server-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=for-the-badge&logo=sqlite&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)

**Protocolos**

![TACACS+](https://img.shields.io/badge/TACACS+_(RFC_8907)-00d4ff?style=for-the-badge&logo=cisco&logoColor=white)
![WebSocket](https://img.shields.io/badge/WebSocket-010101?style=for-the-badge&logo=socketdotio&logoColor=white)
![SSH](https://img.shields.io/badge/SSH-000000?style=for-the-badge&logo=gnometerminal&logoColor=white)
![SSL/TLS](https://img.shields.io/badge/SSL%2FTLS-3A3A3A?style=for-the-badge&logo=letsencrypt&logoColor=white)

---

### Banco de Dados — 7 Tabelas

| Tabela | Descrição | Campos Principais |
|---|---|---|
| `groups` | Grupos de permissão | name, color, permissions (JSON), created_at |
| `switches` | Dispositivos de rede | name, ip_address, model, secret (hash), status, group_id |
| `users` | Usuários do sistema | username, password_hash, privilege_level (0-15), is_admin, group_id |
| `policies` | Políticas de autorização | name, group_id, min_privilege, enabled, priority |
| `policy_commands` | Comandos das políticas | policy_id, action (permit/deny), pattern (regex), order |
| `access_logs` | Logs de acesso | log_type, username, switch_name, ip_address, action, command |
| `server_status` | Estado do servidor TACACS+ | status, port, uptime, total_attempts, successful, failed |

**Sistema de Permissões (JSON por grupo):**
- **7 Páginas:** dashboard, switches, groups, users, policies, logs, terminal
- **6 Ações:** view, create, edit, delete, export, connect

---

### API REST

Todos os endpoints retornam JSON padronizado: `{"success": bool, "timestamp": ISO8601, "message": str, "data": obj}`

| Recurso | Endpoints | Operações |
|---|---|---|
| Dashboard | `GET /api/stats` | Estatísticas gerais |
| Switches | `GET/POST/PUT/DELETE /api/switches` | CRUD completo com validação IP/nome único |
| Grupos | `GET/POST/PUT/DELETE /api/groups` | CRUD + reatribuição de políticas |
| Usuários | `GET/POST/PUT/DELETE /api/users` | CRUD (não-admin só edita a si mesmo) |
| Políticas | `GET/POST/PUT/DELETE /api/policies` | CRUD + toggle enabled |
| Logs | `GET /api/logs` `GET /api/logs/recent` | Paginação, filtros por tipo, limpeza |

**Controle de acesso:** Decorators `@api_permission_required(page, action)` verificam permissões do grupo. Admin (`is_admin=True`) ignora todas as verificações.

---

### Estrutura do Projeto

```
SecurityAuth/
├── app.py                    # Entry point + Flask factory + TACACS+ launcher
├── config.py                 # Configurações (dev/prod/test)
├── models.py                 # 7 modelos ORM (SQLAlchemy)
├── security.py               # Rate limit, headers, sessão, sanitização
├── tacacs_protocol.py        # Protocolo TACACS+ puro (RFC 8907)
├── tacacs_server.py          # Servidor TCP TACACS+ (porta 49)
├── ssh_terminal.py           # Backend SSH via WebSocket (Paramiko)
├── .env                      # Variáveis de ambiente (NÃO versionado)
├── requirements.txt          # Dependências Python
├── deploy.sh                 # Script de deploy automatizado
├── nginx.conf                # Config Nginx de referência
├── migrate_tacacs_net.py     # Migrador XML do TACACS.NET
├── routes/
│   ├── __init__.py            # Blueprints (api_bp, views_bp)
│   ├── api.py                 # Endpoints REST (CRUD completo)
│   └── views.py               # Rotas de páginas HTML + login
├── templates/
│   ├── base.html              # Template base (sidebar, modais, toast)
│   ├── login.html             # Página de login (standalone)
│   ├── dashboard.html         # Dashboard com stats em tempo real
│   ├── switches.html          # CRUD de switches
│   ├── users.html             # CRUD de usuários
│   ├── groups.html            # CRUD de grupos + editor de permissões
│   ├── policies.html          # CRUD de políticas de autorização
│   ├── logs.html              # Visualizador de logs com paginação
│   ├── terminal.html          # Terminal SSH (xterm.js + Socket.IO)
│   └── components/
│       ├── sidebar.html       # Menu lateral com permissões
│       └── modals.html        # Templates de modais reutilizáveis
├── static/
│   ├── css/style.css          # CSS customizado (tema escuro)
│   └── js/app.js              # JavaScript global (modais, toast, API)
└── database/
    ├── schema.sql             # Schema SQL Server (limpo)
    └── schema_sqlserver.sql   # Schema com dados migrados
```

---

### Segurança Implementada

| Camada | Proteção | Detalhes |
|---|---|---|
| **Rede** | Criptografia TACACS+ | Corpo inteiro do pacote encriptado via XOR + MD5 (RFC 8907) |
| **Transporte** | SSL/TLS | HTTPS com TLSv1.2+1.3, ciphers ECDHE, HSTS 2 anos |
| **Aplicação** | Rate Limiting | 5 tentativas/5min, bloqueio 15min por IP |
| **Sessão** | Fingerprint SHA-256 | Detecta hijacking comparando User-Agent + Accept-Language |
| **Sessão** | Timeout inatividade | Logout automático após 60 minutos sem atividade |
| **Input** | Sanitização | Strip, trunca, remove null bytes, validação regex para usernames |
| **Senha** | Hash Werkzeug | generate_password_hash / check_password_hash (scrypt) |
| **CSRF** | Flask-WTF | Token com validade de 1 hora |
| **Cookies** | HttpOnly + SameSite | Lax (Secure em produção), impede acesso JS |
| **Headers** | Segurança HTTP | X-Frame-Options, X-XSS-Protection, CSP, Referrer-Policy |
| **Acesso** | Permissões granulares | JSON por grupo com 7 páginas × 6 ações |

---

### Deploy em Produção

O sistema inclui script `deploy.sh` automatizado para Debian/Ubuntu com 7 etapas:

1. Instala dependências (python3, pip, venv, nginx, certbot)
2. Cria virtualenv e instala requirements + gunicorn + eventlet
3. Configura `.env` com SECRET_KEY aleatório de 64 caracteres
4. Cria serviço systemd (gunicorn + worker eventlet, bind 127.0.0.1:5000)
5. Configura Nginx como proxy reverso com WebSocket pass-through
6. SSL opcional via Let's Encrypt (certbot --nginx)
7. Inicia serviços e exibe status

```bash
# Comandos úteis
systemctl status tacacs-manager     # Status do serviço
systemctl restart tacacs-manager    # Reiniciar
journalctl -u tacacs-manager -f     # Logs em tempo real
```

---

### Dependências Principais

```
Flask==3.0.0
Flask-SQLAlchemy==3.1.1
Flask-Login==0.6.3
Flask-WTF==1.2.1
Flask-Migrate==4.0.5
Flask-SocketIO==5.3.6
Paramiko==3.4.0
pyodbc
gunicorn==21.2.0
eventlet==0.35.1
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

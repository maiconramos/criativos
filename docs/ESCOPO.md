# Escopo do Projeto

> Documento vivo. Atualizar conforme decisões evoluem. Cada decisão deve ter data e razão registrada no log no final.

---

## 1. Visão & Posicionamento

**One-liner provisório (a refinar):**
> Template engine de geração visual, agent-first e API-first, self-hosted — para criadores que querem que seus agentes (Claude Code, Antigravity, Cursor) executem fluxos de imagem/vídeo previamente configurados.

**Categoria:** "n8n / ComfyUI para a era dos agentes."

**Por que existe:** Hoje, quem gera criativo com IA repete o mesmo prompt+inputs N vezes em ChatGPT/Gemini. Templates persistidos + execução via agente eliminam esse retrabalho.

### Tese estratégica de fundo

> **SaaS conversacional está sendo substituído por skills/tools dentro de agentes (Claude Code, OpenWebUI, Cursor, Antigravity).** Não faz mais sentido construir mais um app que compete por atenção humana — a tendência é construir as ferramentas que o agente chama.

**Posicionamento:** não competir com Anthropic/OpenAI no agente. Ser a ferramenta que o agente chama.

**Risco aceito:** estar 6–12 meses à frente do mainstream — o curso vira veículo de evangelização, não só de instalação.

### Visão de ecossistema (médio/longo prazo)

Três produtos do mesmo autor, todos compartilhando convenções (CLI, MCP, API, self-host):

| Produto | Função | Status |
|---|---|---|
| **Criativos** (este projeto) | Geração de imagem/vídeo via agente | Em construção |
| **Multipost** (já em produção) | Publicação em redes sociais via agente | OSS + curso ativo |
| **Futuros** | Outras peças de produção de conteúdo BR | Roadmap mental |

Cada produto tem CLI próprio, MCP server próprio, API REST própria — mas **padrões unificados** (auth, deploy, naming). Agente do usuário pluga em todos.

**Referências de modelo:** n8n, Chatwoot, OpenWebUI, Multipost (próprio autor), ShipFast/IndieKit (boilerplate).

---

## 2. Persona-Alvo

### Persona #1 — Victor, 22 anos
- Sabe gerar imagem do produto X no ChatGPT/Gemini
- Repete inputs e prompts toda vez que precisa
- Já usa Claude Code / agentes
- Quer criar templates uma vez e reusar via agente
- Disposto a pagar VPS + API; semi-técnico

**Personas secundárias (fase 2):** dropshipper solo, gestor de tráfego freelancer, agência pequena, infoprodutor.

---

## 3. Modelo de Negócio

**Playbook:** OSS self-hosted + curso pago — validado pelo autor com Multipost (R$197) e por OpenWebUI.

**Componentes:**
- OSS: código no GitHub, `docker-compose.example.yml` funciona pra quem sabe Docker
- Curso pago (~R$197): instalação guiada, gerador de stack, templates curados, comunidade, aulas de criação de template e integração com agentes
- SaaS hospedado: fase 2/3, opcional

**Canais de aquisição:** Meta Ads (reels de feature), YouTube (vídeos demo), comunidade no Discord/Telegram, SEO via templates da comunidade.

**Diferenças vs Multipost (atenção):**
- Dor é menos óbvia — requer demo concreta no reel
- Tem custo recorrente de API (BYO-key) — endereçar no funil
- Conceito de "template para agente" é novo — precisa educar

---

## 4. Decisões Técnicas Fechadas

| Decisão | Valor | Motivo |
|---|---|---|
| Distribuição | Cloud-only via Docker Compose | Agente precisa de instância sempre-online |
| Linguagem | TypeScript | Boilerplate é TS, MCP precisa de tipos, escala melhor |
| Esqueleto | Boilerplate IndieKit.pro | Já tem auth, multi-tenant, Stripe; acelera meses |
| **Arquitetura de interfaces** | **Core engine (TS lib) + adapters finos (HTTP/MCP/CLI/UI)** | **Evita reimplementar lógica 3x; mantém sync entre interfaces** |
| **Interfaces oficiais** | **HTTP REST + MCP + CLI + Web UI** | **API-first, Agent-first, CLI-first — todos consomem o mesmo core** |
| Auth MCP | Bearer token | Padrão da indústria, baixa fricção |
| MCP transport | HTTP/SSE | Stdio não funciona em cloud |
| Keys de provider | BYO (do usuário) | Self-hosted, sem custo de IA pro autor |
| Providers iniciais | Fal + Replicate | Cobrem ~90% dos casos para anúncio |
| Banco | Postgres (via boilerplate) | Padrão indústria, já vem no esqueleto |
| Storage | Local volume + plug-in R2/S3 opcional | Self-host simples, plug pra escalar |
| Stripe no MVP | Desligado | Self-hosted não cobra |
| Modo local (`npx`) | Não no MVP | Complexidade alta vs valor; demo pública substitui |

---

## 5. O Que Foi Cortado do Projeto Original

- **Electron** — produto é cloud-first agora
- **Cinema Studio** — vira template, não studio
- **Lip Sync Studio** — fora do escopo de anúncio
- **Agent Studio interno (Open-Poe-AI submodule)** — irônico: produto deve SER usado por agentes externos, não ter chat agente interno
- **sd.cpp / Wan2GP local** — fora do escopo cloud
- **Muapi como provider exclusivo** — lock-in removido; vira "mais um provider opcional"
- **JavaScript puro** — migrado pra TypeScript

---

## 6. O Que Será Mantido (e Refatorado)

- **Image Studio** → vira editor de template (não playground)
- **Video Studio** → vira editor de template
- **Workflow Studio** → promovido a **Template Editor** (peça central do produto)
- **Histórico + downloads** → migrado de localStorage pra banco
- **Catálogo de modelos** (curado, ~5–10 modelos relevantes pra anúncio, não 200)

---

## 7. O Que Será Construído do Zero

- **Core Engine (TS library)** — lógica de execução de template, isolada de qualquer interface
  - Pode ser importada como pacote, chamada por adapters HTTP/MCP/CLI/UI
  - Sem dependência de Next/Express/etc; pura função de negócio
- **HTTP REST API** (adapter) — endpoints públicos para uso programático direto
  - Auth: bearer token
  - Documentada (OpenAPI?)
- **MCP Server** (adapter) — HTTP/SSE com bearer auth
  - Tools: `list_templates`, `get_template_schema`, `run_template`, `list_providers`
- **CLI** (adapter) — npx-style, mesma auth/endpoint
  - Comandos: `templates list`, `templates run <name> --input ...`, `providers list`
  - Pra automação local e scripts
- **Provider Abstraction Layer**
  - Camada lógica (nome canônico de modelo)
  - Camada física (mapeamento pra provider concreto)
  - Suporte inicial: Fal, Replicate
- **Template Engine**
  - Template = Workflow + Input Schema + Output Contract
  - Versionamento (modelo mudou → marca template como broken/fallback)
  - Export/import JSON pra portabilidade
- **Auth + Workspace** (single workspace, multi-user opcional, sem orgs)
- **Database schema** (templates, runs, users, api_keys, assets)
- **Stack Generator** (produto separado, paywall do curso)
- **Instância demo pública** (sandbox limitado, hospedado pelo autor)

---

## 8. MVP — Critical Path (na ordem)

1. Migração TypeScript + integração do boilerplate IndieKit (1–2 sem)
2. Provider abstraction layer com Fal + Replicate (1–2 sem)
3. Template engine + schema de inputs (2–3 sem)
4. MCP server HTTP com bearer auth (1 sem)
5. Refator do Workflow Studio → Template Editor UI (1–2 sem)
6. 5 templates seed para criativo de anúncio (1 sem, alta importância)
7. Docker Compose example polido + docs (4–7 dias)
8. Instância demo pública (~3 dias)
9. Landing + captura de email (~3 dias)
10. Beta com 3 pessoas reais (1–2 sem)

**Estimativa total:** 3–4 meses de trabalho focado.

---

## 9. Definition of Done — Gate para Lançar Curso

- [ ] Deploy via `docker-compose.yml` funciona end-to-end pra um não-dev seguindo documentação
- [ ] Fal + Replicate plugados e testados
- [ ] Template Editor funcional na UI
- [ ] MCP HTTP testado com Claude Code real
- [ ] 5 templates seed embutidos
- [ ] **3 beta-testers conseguiram subir e gerar imagem sem intervenção do autor** ← gate real

---

## 10. Templates Seed (5)

> Cada um destes vale mais que metade do código no momento do lançamento. São o que vende o produto na primeira tela.

1. Produto na mão de modelo (lifestyle)
2. Produto em fundo limpo (e-commerce style)
3. Antes/depois (skincare, fitness)
4. Cena de uso (produto sendo usado em contexto)
5. Variação de cor/ângulo do mesmo produto

---

## 11. Métricas Norte

- **TTFV (Time To First Value):** ≤ 15 min do download ao primeiro resultado via agente
- **Setup gratuito (docker-compose) precisa funcionar** — sob risco de perder evangelistas
- **Lista de email** capturada mesmo no fluxo gratuito
- **Conversão OSS → curso:** alvo inicial 3–5%

---

## 12. Decisões Pendentes

- [ ] Nome final do projeto e domínio
- [ ] Frase de 1 linha pro reel (3–5 candidatas)
- [ ] Onde a lista de email entra no fluxo gratuito
- [ ] 3 beta-testers reais identificados (não personas)
- [ ] Política de licença (MIT? AGPL? BSL?)
- [ ] Demo pública: arquitetura (rate-limit, sandbox, custo)
- [ ] Templates seed: prompts/parâmetros concretos
- [ ] Conteúdo detalhado do curso (módulos)

---

## 13. Riscos Mapeados

| Risco | Mitigação |
|---|---|
| Free-path (docker-compose) janky → evangelistas falam mal | Polir docs e tornar funcional, não janky |
| Curso percebido como "só o gerador de stack" | Curso precisa ter 70%+ educação/curadoria, não só utilidade |
| Custo de API + VPS antes do primeiro resultado | Demo pública + free-tier deploy guide |
| Multi-provider quebra templates entre instâncias | Camada lógica/física separada desde o dia 1 |
| Divergência do projeto original (Anil Matcha) | Aceito — projeto bifurca de vez, sem upstream |
| Personas conflitantes diluem produto | Foco MVP só em criativo de anúncio (Victor) |
| **Comfy Cloud MCP fechar o gap (saved workflows by ID)** | **Janela 12–18 meses; MVP em 3–4 meses é prioritário** |
| **Krea AI já tem Nodes+Agent** | **Diferenciador é a combinação 4-pilares + vertical + BR; não feature isolada** |
| **Freepik/Magnific lançar API agent-friendly sobre Spaces** | **Capital deles é alto; ataque o nicho vertical+BR onde eles não vão** |

---

## 14. Análise Competitiva (síntese)

> Pesquisa completa salva em conversa. Documento vivo — atualizar quando concorrentes mudam.

### Smoking gun confirmado
**ComfyUI MCP** (o mais maduro do espaço) tem limitação documentada explicitamente:
> *"Workflows não podem ser executados por ID salvo — agente reconstrói do zero a cada vez"*

Isso é literalmente o que o template engine persistido resolve. Argumento de marketing direto.

### Posição única (4 pilares + vertical + BR)
Ninguém combina:
1. Self-host (Docker)
2. Template persistido (callable por agente por ID/nome)
3. MCP nativo (não wrapper de provedor)
4. Multi-provider abstrato
5. Vertical anúncio
6. BR-first

### Concorrentes mais perigosos (em ordem)
1. **Krea AI** — já tem Nodes + Nodes Agent. SaaS fechado, sem MCP/self-host/BYO-key/BR. Risco de feature parity.
2. **Comfy Cloud (oficial)** — tem MCP maduro, vai fechar o gap de saved workflows. 12–18 meses.
3. **Freepik/Magnific Spaces** — visível, capital alto, mas sem agente/API friendly hoje.
4. **InvokeAI** — único OSS sério com workflow editor. Sem MCP. Pode ganhar via comunidade.

### Categorias mapeadas
- **Flow/Workflow SaaS:** Magnific/Spaces, Krea, Leonardo Elements, Flora, Segmind Pixelflow, Recraft
- **Workflow OSS:** ComfyUI (+ Comfy Cloud, RunComfy, ComfyDeploy), InvokeAI, Fooocus
- **Vertical anúncio SaaS:** AdCreative.ai ($39+), Pencil, Predis, Omneky, Smartly.io, Canva Magic, Adobe Firefly, Creatify, Arcads
- **Template-API estático (não generativo):** Bannerbear, Placid, Abyssale, Templated.io, Orshot (único com BYO-key)
- **Provider layer:** Replicate, Fal.ai, Runware (cost leader), Together, Muapi, WaveSpeedAI
- **MCPs existentes:** mcp-replicate, mcp-fal, comfy-cloud-mcp, comfyui-mcp (artokun), mcp-fooocus-api

### Mercado BR
- **Vazio.** Nenhum SaaS BR estabelecido na categoria.
- Hotmart vende "Agentes de IA" como infoproduto = sinal de demanda madura.
- Cursos BR de ComfyUI existem (Udemy, Domestika); ninguém combina tool+curso+comunidade vertical anúncio.

---

## 15. Log de Decisões

| Data | Decisão | Razão |
|---|---|---|
| 2026-05-13 | Foco MVP em criativo para anúncio (não dark channel / agência) | Maior dor imediata, ciclo de feedback curto, público pagante |
| 2026-05-13 | Cloud-only via Docker Compose, sem modo local | Agente precisa de instância sempre-online |
| 2026-05-13 | TypeScript desde o dia 1 | Boilerplate é TS, MCP precisa de tipos |
| 2026-05-13 | Boilerplate IndieKit como esqueleto | Já tem auth/multi-tenant, acelera meses |
| 2026-05-13 | BYO-key, Stripe off no MVP | Self-hosted, sem custo de IA pro autor |
| 2026-05-13 | Eletron morre | Cloud-first, agent-first |
| 2026-05-13 | Cinema/LipSync/Agent studios morrem | Fora do escopo de anúncio |
| 2026-05-13 | Multi-provider (Fal + Replicate); Muapi vira opcional | Remover lock-in é diferencial central |
| 2026-05-13 | Stack generator paywall do curso (R$197) | Playbook validado por Multipost / OpenWebUI |
| 2026-05-13 | Arquitetura: core engine + adapters (HTTP/MCP/CLI/UI) | Evita reimplementar lógica em cada interface; mantém sync |
| 2026-05-13 | API-first + Agent-first + CLI-first como princípios oficiais | Produto deve ser consumível por agente, script ou humano com mesma fluidez |
| 2026-05-13 | Visão de ecossistema (Criativos + Multipost + futuros) com convenções unificadas | Cada produto reforça os outros; agente do usuário pluga em todos |
| 2026-05-13 | Aceito risco de "à frente do tempo" (6–12 meses) | Curso vira evangelização; moat por antecipação da curva |
| 2026-05-13 | Posicionamento confirmado por análise competitiva | Espaço "self-host + template persistido + MCP + multi-provider + vertical anúncio + BR" está vazio |
| 2026-05-13 | Janela de 12–18 meses é restrição real | MVP em 3–4 meses é prioritário antes que Comfy/Krea fechem gaps |

# iccmonit â€” INEMA Claude Monitor

TUI em Python que monitora **todas as sessÃµes Claude Code ativas** na mÃ¡quina em tempo real, com cota 5h/7d, mÃ©tricas por sessÃ£o e chat embutido via OAuth.

Pensado para rodar em um **terminal separado** ao lado das suas sessÃµes Claude Code, dando visÃ£o consolidada de:

- SessÃµes ativas (busy / idle), modelo em uso, % do contexto consumido
- Cota da API: janela de 5 horas, semanal global e semanal Sonnet (separada)
- Tamanho do `CLAUDE.md`, da memÃ³ria do projeto, agentes lanÃ§ados e skills invocadas
- Chat com Claude (Haiku por padrÃ£o) ciente do estado do painel

VersÃ£o atual: **v1.17.11**

![iccmonit em execuÃ§Ã£o â€” painel de cota, sessÃµes e chat focado](docs/img/screenshot.jpg)

---

## SumÃ¡rio

- [Requisitos](#requisitos)
- [InstalaÃ§Ã£o](#instalaÃ§Ã£o)
- [Uso](#uso)
  - [Modo terminal](#modo-terminal)
  - [Modo web](#modo-web)
  - [Atalhos](#atalhos-de-teclado)
- [PainÃ©is da TUI](#painÃ©is-da-tui)
- [Fontes de dados](#fontes-de-dados)
- [ConfiguraÃ§Ã£o](#configuraÃ§Ã£o--configjson)
- [Arquitetura](#arquitetura)
- [Versionamento](#versionamento)
- [Roadmap](#roadmap)
- [SoluÃ§Ã£o de problemas](#soluÃ§Ã£o-de-problemas)
- [Por que nÃ£o usar X?](#por-que-nÃ£o-usar-x)
- [LicenÃ§a](#licenÃ§a)

---

## Requisitos

- **Python 3.10+**
- **Claude Code instalado e logado** (precisa de `~/.claude/.credentials.json` vÃ¡lido)
- **Statusline padrÃ£o do Claude Code rodando** â€” Ã© ele que popula `/tmp/cc_limits_<uid>.json` com a cota oficial da API. Sem isso o painel de cota fica vazio.
- Sistema operacional **Linux** ou **macOS** (testado no Linux). O `tac` Ã© usado para ler transcripts de trÃ¡s pra frente â€” em macOS tem o equivalente.

## InstalaÃ§Ã£o

```bash
git clone git@github.com:inematds/iccmonit.git
cd iccmonit
chmod +x start.sh
```

DependÃªncias (instaladas automaticamente pelo `start.sh` se faltarem):

```bash
pip install textual anthropic --break-system-packages           # modo terminal
pip install textual-serve --break-system-packages               # modo web (extra)
```

## Uso

### Modo terminal

```bash
./start.sh
```

Abre a TUI no terminal atual. Ideal para rodar lado-a-lado com as sessÃµes Claude Code.

### Modo web

```bash
./start.sh web                       # 127.0.0.1:8000     â€” apenas local (padrÃ£o)
./start.sh web 9000                  # 127.0.0.1:9000     â€” porta custom
./start.sh web 8000 lan              # <ip-da-LAN>:8000   â€” outras mÃ¡quinas da rede
./start.sh web 8000 192.168.1.50     # IP especÃ­fico
```

Usa o pacote **`textual-serve`** para empacotar a TUI atual num WebSocket + xterm.js no navegador, sem mudanÃ§a de cÃ³digo. O servidor (`serve.py`) abre o navegador automaticamente.

**Modos de bind:**

| Modo | Bind | Quem acessa |
|------|------|-------------|
| padrÃ£o | `127.0.0.1` | SÃ³ o prÃ³prio host |
| `lan`  | IP da LAN auto-detectado (rota pro `8.8.8.8`) | Qualquer mÃ¡quina na sub-rede |
| `<ip>` | IP fornecido | Quem rotear atÃ© esse IP |

> âš  **Sem autenticaÃ§Ã£o.** No modo `lan`, qualquer um na rede com o IP+porta acessa o monitor â€” incluindo o chat OAuth conectado Ã  sua conta. Use sÃ³ em rede de confianÃ§a. Pra acesso atravÃ©s da internet, prefira **SSH tunnel** em vez de expor a porta:
>
> ```bash
> ssh -L 8000:localhost:8000 usuario@maquina
> ```
>
> Bindar em `0.0.0.0` estÃ¡ **bloqueado** porque o `textual-serve` embute o host de bind no websocket e o navegador rejeita conexÃ£o WS pra `0.0.0.0`. Ãštil pra:

- Acompanhar de outra mÃ¡quina na rede local (acesse `http://<ip-da-maquina>:8000`)
- Manter o monitor visÃ­vel num browser sem ocupar terminal
- Compartilhar visÃ£o temporÃ¡ria com colegas na mesma rede

### Ajuda

```bash
./start.sh -h    # mostra os modos disponÃ­veis
```

### Atalhos de teclado

| Tecla | AÃ§Ã£o |
|-------|------|
| `r`   | Refresh manual |
| `a`   | Alterna entre **SessÃµes ativas** â†” **todas** (vivas + mortas) |
| `p`   | Liga/desliga o painel **Processos** (top N da mÃ¡quina) |
| `v`   | Liga/desliga o painel **ServiÃ§os** (Docker + systemd boot) |
| `s`   | Quando Processos visÃ­vel: alterna ordenaÃ§Ã£o **CPU% â†” RAM%** |
| `,`   | Encolhe a coluna esquerda em 5% (chat fica maior) |
| `.`   | Aumenta a coluna esquerda em 5% (chat fica menor) |
| `=`   | Reseta a divisÃ£o em 50/50 |
| `1`   | Abre **Cota** em fullscreen modal |
| `2`   | Abre **MÃ¡quina** em fullscreen |
| `3`   | Abre **Processos** em fullscreen |
| `4`   | Abre **SessÃµes** em fullscreen |
| `5`   | Abre **ServiÃ§os** em fullscreen (Docker completo + systemd boot completo) |
| **click** no tÃ­tulo de uma seÃ§Ã£o | Mesmo efeito do `1`-`4` correspondente |
| `Esc` ou `q` (no modal) | Fecha o modal |
| `q`   | Sair |

Auto-refresh a cada **10 segundos** por padrÃ£o (configurÃ¡vel em `config.json` via `refresh_interval_seconds`). A coleta (lÃª transcripts grandes, roda `nvidia-smi`, `docker ps`, etc.) acontece numa **worker thread** â€” a TUI nÃ£o congela e os valores antigos ficam na tela atÃ© o novo lote chegar. O subtitle do header mostra **"atualizado hÃ¡ Xs"** ticando a cada segundo.

Chat tambÃ©m Ã© assÃ­ncrono: ao enviar, a mensagem aparece na hora com `aguardando claude...` enquanto a API responde em background. Mandar nova mensagem antes da anterior chegar Ã© bloqueado com aviso.

---

## PainÃ©is da TUI

A TUI tem layout em duas colunas (50/50 por padrÃ£o, redimensionÃ¡vel com `,` `.` `=`):

- **Esquerda** â€” empilha verticalmente: Cota â†’ MÃ¡quina â†’ Processos (opcional) â†’ SessÃµes
- **Direita** â€” Chat ocupando toda a altura

Limites de redimensionamento: 20% / 80%. NÃ£o fica tudo de um lado sÃ³.

### 1. Cota (topo)

Barra horizontal por janela:

| Linha | Significado |
|-------|-------------|
| `5h`  | Janela rolante de 5 horas â€” todos os modelos |
| `7d`  | Janela semanal â€” todos os modelos |
| `7dâ™ª` | Janela semanal **Sonnet** â€” limite separado, importante monitorar |

Cor da barra segue os thresholds de `quota_pct` em `config.json`. Se a barra aparecer "aguardando API..." Ã© porque o cache `/tmp/cc_limits_<uid>.json` estÃ¡ vazio ou velho (> 2 min).

### 2. MÃ¡quina

Uso de hardware do servidor â€” barras coloridas pelos thresholds de `system_pct`:

| Linha | Mostra |
|-------|--------|
| `CPU`  | `usage_pct` (load1 / cores) + load 1m/5m/15m + nÂº de cores |
| `RAM`  | `used / total MB` + swap usado |
| `Disk` | `used / total GB` + livre |
| `GPU`  | `nvidia-smi`: utilizaÃ§Ã£o %, VRAM, temperatura. Linha some se nÃ£o houver GPU NVIDIA. |

CPU usa `os.getloadavg()` (nÃ£o `psutil`), RAM lÃª `/proc/meminfo`, GPU Ã© opcional via `nvidia-smi`. Sem dependÃªncias extras.

### 3. Processos (opcional â€” tecla `p`)

Top N processos da mÃ¡quina, ordenados por **%CPU** ou **%RAM** (alterna com `s`). Default: oculto.

| Coluna | Mostra |
|--------|--------|
| `PID` | identificador do processo |
| `User` | dono (truncado em 10 chars) |
| `CPU%` | % de CPU (cor pelos thresholds de `system_pct`) |
| `RAM%` | % de RAM (mesma escala de cor) |
| `Tempo` | etime (`HH:MM:SS` ou `D-HH:MM:SS`) |
| `Comando` | comm truncado em 40 chars |

Coleta via `ps -eo pid,user,pcpu,pmem,etime,comm --sort=-%cpu` (sem dependÃªncia extra). Ãštil pra:

- Saber **quais sessÃµes `claude` estÃ£o consumindo CPU/RAM** (cada PID aparece separado)
- Detectar processos pesados quando o painel MÃ¡quina ficar ðŸŸ¡/ðŸ”´
- Ver `ollama`, GPU workers, dev servers etc. na mesma tela

ConfigurÃ¡vel: `top_processes_n` no `config.json` (padrÃ£o `10`). O painel sÃ³ polla `ps` quando estÃ¡ visÃ­vel â€” nÃ£o tem custo quando oculto.

### 4. ServiÃ§os (opcional â€” tecla `v`)

Resumo de **Docker** e **systemd boot** num painel sÃ³. Default: oculto.

**Inline mostra sÃ³ o que importa:**

```
Docker   10/44 containers ativos
  âœ“ imkt4-postgres-1     Up 4 days     0.0.0.0:5432->5432/tcp
  âœ“ openclaw-ollama      Up 4 days     0.0.0.0:11434->11434/tcp
  âœ“ open-webui           Up 4 days     0.0.0.0:3000->8080/tcp
  ... +7 ativos Â· use 5 pra ver tudo (incl. parados)
Boot     33/102 systemd .service habilitados rodando  (abra modal pra ver lista)
```

**Modal (`5`)** mostra a lista completa: todos os containers (incluindo parados, com cÃ³digo de saÃ­da) e todas as services systemd habilitadas com estado atual (`running`, `dead`, `exited`, `failed`).

Coleta:
- Docker via `docker ps -a` â€” se `docker` nÃ£o estiver no `$PATH`, mostra "indisponÃ­vel"
- Boot via 2 chamadas a `systemctl` (1Ã— `list-unit-files --state=enabled`, 1Ã— `list-units --all` para o estado) â€” sem polling unitÃ¡rio, leve
- SÃ³ roda quando o painel estÃ¡ visÃ­vel (zero custo quando oculto)

### 5. SessÃµes (ativas ou todas)

Tabela com uma linha por sessÃ£o Claude Code. Por padrÃ£o mostra sÃ³ sessÃµes com PID alive; pressione **`a`** para incluir as mortas (sessÃµes cujo arquivo em `~/.claude/sessions/*.json` ainda existe mas o processo terminou). Mortas aparecem em cinza com status `morta`. Colunas:

| Coluna | DescriÃ§Ã£o |
|--------|-----------|
| **Projeto** | Nome do diretÃ³rio de trabalho (truncado em 20 chars) |
| **Status** | `idle` (verde) ou `busy` (amarelo) |
| **Modelo** | `opus`, `sonnet`, `haiku` ou outro |
| **Ctx%** | % da janela de 1M tokens em uso. Soma `input + cache_read + cache_creation + output` |
| **CLAUDE.md** | Tamanho do `CLAUDE.md` no `cwd` da sessÃ£o |
| **Mem** | `<n>f/<x>k` â€” nÃºmero de arquivos e KB totais em `memory/` |
| **Agts** | Quantidade de chamadas Ã  tool `Agent` no transcript |
| **Skls** | Skills distintas invocadas (tool `Skill`) |
| **Update** | Tempo desde o Ãºltimo update (`12s`, `3m`, `1h`) |

Cada mÃ©trica recebe cor azul/verde/amarelo/vermelho conforme thresholds em `config.json`.

### 6. Chat (lateral)

Chat embutido com Claude com **dois modos**:

**Modo geral** (padrÃ£o) â€” recebe o estado completo do painel como system prompt:

- "Qual sessÃ£o tÃ¡ usando mais contexto?"
- "Quanto falta pra cota Sonnet?"
- "Algum projeto com memÃ³ria crÃ­tica?"

**Modo focado** â€” selecione uma linha da tabela (â†‘/â†“ + **Enter**) e o chat carrega o transcript daquela sessÃ£o como contexto. AÃ­ dÃ¡ pra perguntar sobre o trabalho especÃ­fico:

- "O que essa sessÃ£o tÃ¡ fazendo agora?"
- "Qual foi o Ãºltimo erro?"
- "Resume o que foi feito atÃ© agora."
- "Por que ele tomou essa decisÃ£o?"

> O chat Ã© um **observador analÃ­tico** â€” nÃ£o interage com o agente daquela sessÃ£o, sÃ³ lÃª o transcript. Pra interaÃ§Ã£o remota real, ver V2 no roadmap.

#### Comandos do chat

| Comando | AÃ§Ã£o |
|---------|------|
| `/help`   | Ajuda completa (TUI + chat + skills + roadmap) |
| `/diag`   | DiagnÃ³stico do painel â€” lista cota e sessÃµes em alerta (ðŸŸ¡/ðŸ”´) com sugestÃµes |
| `/skills` | 3 skills Ãºteis pra rodar nas sessÃµes focadas (statusline / memory-audit / handoff) |
| `/clear`  | Volta ao modo geral (limpa o foco e o histÃ³rico) |
| `/log`    | Mostra as Ãºltimas 20 linhas do log de chat |
| `/where`  | Imprime os caminhos do log, config e diretÃ³rio de projects |
| `/docker` | sem args: lista containers rodando Â· `/docker <nome> start\|stop\|restart\|logs` age direto |
| `/fork`   | *(roadmap V2)* â€” abrir nova sessÃ£o Claude Code continuando a focada |

Trocar de foco tambÃ©m limpa o histÃ³rico do chat â€” pra nÃ£o misturar conversas.

#### DiagnÃ³stico do painel â€” `/diag`

Roda regras locais sobre os thresholds do `config.json` e devolve sÃ³ o que estÃ¡ em **ðŸŸ¡ yellow** ou **ðŸ”´ red**:

- Cotas (5h, 7d, 7d Sonnet) em alerta
- SessÃµes com `Ctx%` alto (risco de auto-compaction)
- SessÃµes com memÃ³ria ou `CLAUDE.md` inchados
- SessÃµes com muitos agentes lanÃ§ados

Quando algo aparece, sugere a skill apropriada do `skillmanager3x` (`session-handoff`, `memory-audit`, etc).

#### Skills Ãºteis â€” `/skills`

Lista as 3 skills do [skillmanager3x](https://github.com/inematds/skillmanager3x) com **quando usar cada uma**:

| Skill | Quando rodar |
|-------|--------------|
| **session-statusline** | Checkpoint operacional rÃ¡pido durante o trabalho. Triggers: "onde estamos", "checkpoint", "organiza a sessÃ£o". |
| **memory-audit**       | Quando o painel mostra memÃ³ria ðŸŸ¡/ðŸ”´ ou CLAUDE.md grande. Triggers: "analise a memÃ³ria", "memÃ³ria inchada", "limpa memÃ³ria". |
| **session-handoff**    | Antes de `/clear` ou quando ctx ðŸ”´. Gera resumo pra um agente fresco continuar. Triggers: "vou dar /clear", "handoff", "encerrar sessÃ£o". |

Como rodar: na sessÃ£o Claude Code alvo, digite o trigger ou `/<skill-name>`.

#### Log de chat

Cada turno (vocÃª â†’ claude) e cada erro sÃ£o gravados em `chat.log` no diretÃ³rio do projeto. Inclui:

- timestamp, role (user/claude), foco (`[geral]` ou `[focus=<8 chars>]`), e texto
- em caso de erro: status code e body cru da resposta da API

Ãštil pra debugar 401 (OAuth expirado/invÃ¡lido), rate limit, payload invÃ¡lido, etc. O arquivo estÃ¡ no `.gitignore`.

Modelo padrÃ£o: `claude-haiku-4-5-20251001` (configurÃ¡vel em `config.json` â†’ `chat.model`). AutenticaÃ§Ã£o via OAuth do Claude Code (token em `~/.claude/.credentials.json`).

---

## Fontes de dados

| MÃ©trica | Fonte |
|---|---|
| SessÃµes ativas | `~/.claude/sessions/*.json` â€” um arquivo por PID, filtrado por processo vivo (`os.kill(pid, 0)`) |
| Modelo, contexto %, custo | Ãšltima mensagem `assistant` com `usage` no transcript JSONL **ativo** em `~/.claude/projects/<encoded-cwd>/`. O `sessionId` em `session.json` congela no boot â€” apÃ³s `/clear` o Claude Code cria um JSONL novo e o monitor segue o de `mtime` mais recente |
| Cota 5h / 7d / 7d-Sonnet | `/tmp/cc_limits_<uid>.json` â€” cache do statusline (fonte oficial via API Anthropic) |
| `CLAUDE.md` | `stat` do arquivo em `<cwd>/CLAUDE.md` |
| MemÃ³ria | Conta arquivos e soma bytes de `~/.claude/projects/<encoded-cwd>/memory/` |
| Agentes lanÃ§ados | Tool uses `name == "Agent"` no transcript (acumulado na sessÃ£o) |
| Skills invocadas | Tool uses `name == "Skill"` no transcript (distintas) |
| Chat OAuth | `~/.claude/.credentials.json` â†’ `claudeAiOauth.accessToken`. Header: `anthropic-beta: oauth-2025-04-20` |

> A cota **nÃ£o** Ã© recalculada localmente como `ccm`/`ccusage` fazem â€” Ã© lida do cache que o prÃ³prio Claude Code mantÃ©m com os valores oficiais da API. Isso evita divergÃªncia.

### Encoding do `cwd`

Barras e dois pontos viram hÃ­fens, com hÃ­fen prefixado:

```
/home/user/projetos/foo  â†’  -home-user-projetos-foo
```

O monitor tenta ambas as variaÃ§Ãµes (com e sem hÃ­fen prefixado) ao localizar transcript e diretÃ³rio de memÃ³ria.

---

## ConfiguraÃ§Ã£o â€” `config.json`

EditÃ¡vel sem tocar no cÃ³digo. Estrutura:

### `thresholds`

Limites de cor por mÃ©trica. Cada mÃ©trica tem 4 nÃ­veis: `blue` â‰¤ `green` â‰¤ `yellow` â‰¤ `red`.

| MÃ©trica | O que mede | Defaults (b/g/y/r) |
|---------|------------|--------------------|
| `quota_pct`           | % da cota usada (5h, 7d, 7d-Sonnet) | 20 / 60 / 85 / 95 |
| `context_pct`         | % da janela de contexto da sessÃ£o. >90% = risco de auto-compaction | 10 / 50 / 75 / 90 |
| `memory_kb`           | Tamanho total da memÃ³ria do projeto (KB). >600 = considere `/memory-audit` | 0 / 100 / 300 / 600 |
| `memory_files`        | NÃºmero de arquivos de memÃ³ria. >35 = fragmentaÃ§Ã£o | 0 / 10 / 20 / 35 |
| `claude_md_bytes`     | Tamanho do `CLAUDE.md`. >40KB = pode impactar contexto | 0 / 8192 / 20480 / 40960 |
| `agents_per_session`  | Agentes lanÃ§ados na sessÃ£o. >30 = sessÃ£o muito pesada | 0 / 5 / 15 / 30 |
| `cost_per_session_usd`| Custo acumulado em USD. >$20 = revisar eficiÃªncia | 0 / 2 / 8 / 20 |

Esquema visual:

| Cor | Significado |
|-----|-------------|
| ðŸ”µ Azul | Muito baixo / ocioso |
| ðŸŸ¢ Verde | Normal / saudÃ¡vel |
| ðŸŸ¡ Amarelo | AtenÃ§Ã£o |
| ðŸ”´ Vermelho | CrÃ­tico / alerta |

### `alerts`

Habilita/desabilita mensagens de alerta individualmente:

| Chave | PadrÃ£o | Mensagem |
|-------|--------|----------|
| `quota_5h_red`  | enabled  | âš  Cota 5h crÃ­tica! |
| `quota_7d_red`  | enabled  | âš  Cota semanal crÃ­tica! |
| `context_red`   | enabled  | âš  Contexto alto â€” risco de compaction |
| `memory_red`    | enabled  | âš  MemÃ³ria excessiva â€” rode `/memory-audit` |
| `claude_md_red` | disabled | âš  CLAUDE.md muito grande |
| `cost_red`      | enabled  | âš  SessÃ£o cara |

### `chat`

| Campo | PadrÃ£o | DescriÃ§Ã£o |
|-------|--------|-----------|
| `model`      | `claude-haiku-4-5-20251001` | Modelo do chat embutido |
| `max_tokens` | `1024` | Tamanho mÃ¡ximo da resposta |

### Outros

| Campo | PadrÃ£o | DescriÃ§Ã£o |
|-------|--------|-----------|
| `title` | `INEMA Claude Monitor` | TÃ­tulo do header (a versÃ£o Ã© appendada automaticamente) |
| `refresh_interval_seconds` | `10` | Intervalo de auto-refresh em segundos |

---

## Arquitetura

```
iccmonit/
â”œâ”€â”€ monitor.py     # TUI principal (Textual) â€” V1
â”œâ”€â”€ serve.py       # Wrapper de modo web via textual-serve
â”œâ”€â”€ start.sh       # Entrypoint: terminal e modo web
â”œâ”€â”€ config.json    # Thresholds, alertas, chat, refresh
â”œâ”€â”€ README.md      # Este arquivo
â”œâ”€â”€ CLAUDE.md      # InstruÃ§Ãµes do projeto pra Claude Code
â”œâ”€â”€ .gitignore
â””â”€â”€ docs/
    â”œâ”€â”€ img/         # Screenshots
    â”œâ”€â”€ PLAN.md      # Plano original e decisÃµes de arquitetura
    â””â”€â”€ RESEARCH.md  # Pesquisa das fontes de dados do Claude Code
```

### Componentes do `monitor.py`

| SÃ­mbolo | Papel |
|---------|-------|
| `load_sessions()` | LÃª `~/.claude/sessions/*.json` e filtra por PID vivo |
| `get_transcript_usage()` | LÃª Ãºltima `assistant.usage` do transcript via `tac` |
| `get_session_extras()` | Calcula tamanho de `CLAUDE.md`, memÃ³ria, contagem de agentes/skills |
| `load_quota()` | LÃª `/tmp/cc_limits_<uid>.json` (descarta se > 120s) |
| `get_oauth_token()` | Extrai `accessToken` do `.credentials.json` |
| `QuotaBar` | Widget com barras de cota |
| `SessionTable` | DataTable com sessÃµes |
| `ChatPane` | RichLog + Input â€” chat com `system` montado a cada mensagem com snapshot do painel |
| `MonitorApp` | App Textual; auto-refresh via `set_interval` |

---

## Versionamento

Constante `VERSION` em `monitor.py` no formato **`v1.xx.yy`**:

- **major (`1`)** â€” sÃ³ muda na transiÃ§Ã£o para V2
- **`xx`** â€” incrementa a cada **recurso novo** (sequencial: `01`, `02`, ...)
- **`yy`** â€” incrementa a cada **bug fix** (sequencial dentro da major)
- **`yy` NÃƒO reinicia** quando `xx` sobe â€” sÃ³ zera junto com `xx` na transiÃ§Ã£o de major

Exemplos:
- `v1.00.00` â†’ estado inicial
- `v1.01.00` â†’ primeiro recurso novo (nenhum bug fix ainda)
- `v1.01.02` â†’ apÃ³s dois bugs fixados dentro do recurso `01`
- `v1.02.02` â†’ prÃ³ximo recurso (`yy` mantÃ©m em `02` porque nenhum novo bug foi corrigido)
- `v1.02.03` â†’ bug fix sobre `v1.02.02`
- `v2.00.00` â†’ mudanÃ§a de major zera tudo

A versÃ£o Ã© exibida no tÃ­tulo da janela TUI (header).

---

## Roadmap

- **V1** *(atual)* â€” painel de sessÃµes + cota + chat geral + chat focado em sessÃ£o (read-only) + modo web
- **V2** *(planejado)* â€” interaÃ§Ã£o remota com sessÃ£o ativa: injetar prompts numa sessÃ£o como se estivesse no terminal original. Mecanismo a investigar:
  - stdin do processo Claude Code
  - arquivo de IPC
  - ou `claude --resume <sessionId>` num processo paralelo (forka, nÃ£o dirige a sessÃ£o original)
  - sem API oficial â€” requer pesquisa

---

## SoluÃ§Ã£o de problemas

| Sintoma | Causa provÃ¡vel | Como resolver |
|---------|----------------|---------------|
| Cota mostra `aguardando API...` | `/tmp/cc_limits_<uid>.json` ausente ou > 2 min | Confirme que o statusline padrÃ£o do Claude Code estÃ¡ rodando â€” ele popula esse cache |
| Chat indisponÃ­vel | Token OAuth nÃ£o encontrado | FaÃ§a login novamente no Claude Code e confirme que `~/.claude/.credentials.json` tem `claudeAiOauth.accessToken` |
| Chat retorna `401` | Token OAuth expirado, revogado, ou cota da janela esgotada | Use `/log` no chat pra ver o body do erro. Se expirado, faÃ§a `claude logout` + `claude login`. Se cota, espere a janela renovar (vide painel de cota). |
| Tabela vazia | Nenhuma sessÃ£o Claude Code ativa | Abra uma sessÃ£o `claude` em outro terminal e dÃª `r` para refresh |
| `Ctx%` zerado | Transcript ainda sem mensagem `assistant` com `usage` | Use a sessÃ£o um pouco â€” depois da primeira resposta o valor aparece |
| Modo web nÃ£o abre | Porta ocupada | Tente outra porta: `./start.sh web 9001` |
| Modo web abre o "demo" do Textual | VersÃ£o antiga sem `textual-serve` | Atualize: `git pull` (a partir de v1.01.01 usa `textual-serve` separado) |
| Modo web mostra sÃ³ o logo "iccmonit" e nÃ£o carrega a TUI | VersÃ£o `v1.01.01` bindando em `0.0.0.0` (websocket invÃ¡lido no navegador) | Atualize pra `v1.01.02`+ â€” bind virou `127.0.0.1` |
| ApÃ³s `/clear` o `Ctx%` continua alto | VersÃ£o â‰¤ `v1.02.00` lia o JSONL antigo (sessionId congelado em `session.json`) | Atualize pra `v1.02.01`+ â€” passa a seguir o JSONL mais recente (`mtime`) |
| Erro `ImportError: textual` | DependÃªncias nÃ£o instaladas | Rode `pip install textual anthropic --break-system-packages` ou simplesmente `./start.sh` (instala automaticamente) |

---

## Por que nÃ£o usar X?

- **`ccm` / claude-monitor** â€” calcula cota a partir dos JSONL locais e diverge da cota real da API. Aqui usamos o cache oficial.
- **`ccusage`** â€” ferramenta de auditoria/relatÃ³rio, nÃ£o tempo real.
- **`claude-pulse`** â€” o `statusline-command.sh` existente jÃ¡ cobre o mesmo escopo.

---

## LicenÃ§a

MIT.

---

Projeto adaptado da comunidade INEMA TDS - Marca Arckhe
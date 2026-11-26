ï»¿# iccmonit ê¢â¬â ARCKHÉ Claude Monitor

TUI em Python que monitora **todas as sessêÂµes Claude Code ativas** na mêÂ¡quina em tempo real, com cota 5h/7d, mêÂ©tricas por sessêÂ£o e chat embutido via OAuth.

Pensado para rodar em um **terminal separado** ao lado das suas sessêÂµes Claude Code, dando visêÂ£o consolidada de:

- SessêÂµes ativas (busy / idle), modelo em uso, % do contexto consumido
- Cota da API: janela de 5 horas, semanal global e semanal Sonnet (separada)
- Tamanho do `CLAUDE.md`, da memêÂ³ria do projeto, agentes lanêÂ§ados e skills invocadas
- Chat com Claude (Haiku por padrêÂ£o) ciente do estado do painel

VersêÂ£o atual: **v1.17.11**

![iccmonit em execuêÂ§êÂ£o ê¢â¬â painel de cota, sessêÂµes e chat focado](docs/img/screenshot.jpg)

---

## SumêÂ¡rio

- [Requisitos](#requisitos)
- [InstalaêÂ§êÂ£o](#instalaêÂ§êÂ£o)
- [Uso](#uso)
  - [Modo terminal](#modo-terminal)
  - [Modo web](#modo-web)
  - [Atalhos](#atalhos-de-teclado)
- [PainêÂ©is da TUI](#painêÂ©is-da-tui)
- [Fontes de dados](#fontes-de-dados)
- [ConfiguraêÂ§êÂ£o](#configuraêÂ§êÂ£o--configjson)
- [Arquitetura](#arquitetura)
- [Versionamento](#versionamento)
- [Roadmap](#roadmap)
- [SoluêÂ§êÂ£o de problemas](#soluêÂ§êÂ£o-de-problemas)
- [Por que nêÂ£o usar X?](#por-que-nêÂ£o-usar-x)
- [LicenêÂ§a](#licenêÂ§a)

---

## Requisitos

- **Python 3.10+**
- **Claude Code instalado e logado** (precisa de `~/.claude/.credentials.json` vêÂ¡lido)
- **Statusline padrêÂ£o do Claude Code rodando** ê¢â¬â êÂ© ele que popula `/tmp/cc_limits_<uid>.json` com a cota oficial da API. Sem isso o painel de cota fica vazio.
- Sistema operacional **Linux** ou **macOS** (testado no Linux). O `tac` êÂ© usado para ler transcripts de trêÂ¡s pra frente ê¢â¬â em macOS tem o equivalente.

## InstalaêÂ§êÂ£o

```bash
git clone git@github.com:linnoff88-dev/iccmonit.git
cd iccmonit
chmod +x start.sh
```

DependêÂªncias (instaladas automaticamente pelo `start.sh` se faltarem):

```bash
pip install textual anthropic --break-system-packages           # modo terminal
pip install textual-serve --break-system-packages               # modo web (extra)
```

## Uso

### Modo terminal

```bash
./start.sh
```

Abre a TUI no terminal atual. Ideal para rodar lado-a-lado com as sessêÂµes Claude Code.

### Modo web

```bash
./start.sh web                       # 127.0.0.1:8000     ê¢â¬â apenas local (padrêÂ£o)
./start.sh web 9000                  # 127.0.0.1:9000     ê¢â¬â porta custom
./start.sh web 8000 lan              # <ip-da-LAN>:8000   ê¢â¬â outras mêÂ¡quinas da rede
./start.sh web 8000 192.168.1.50     # IP especêÂ­fico
```

Usa o pacote **`textual-serve`** para empacotar a TUI atual num WebSocket + xterm.js no navegador, sem mudanêÂ§a de cêÂ³digo. O servidor (`serve.py`) abre o navegador automaticamente.

**Modos de bind:**

| Modo | Bind | Quem acessa |
|------|------|-------------|
| padrêÂ£o | `127.0.0.1` | SêÂ³ o prêÂ³prio host |
| `lan`  | IP da LAN auto-detectado (rota pro `8.8.8.8`) | Qualquer mêÂ¡quina na sub-rede |
| `<ip>` | IP fornecido | Quem rotear atêÂ© esse IP |

> ê¢Å¡Â  **Sem autenticaêÂ§êÂ£o.** No modo `lan`, qualquer um na rede com o IP+porta acessa o monitor ê¢â¬â incluindo o chat OAuth conectado êÂ  sua conta. Use sêÂ³ em rede de confianêÂ§a. Pra acesso atravêÂ©s da internet, prefira **SSH tunnel** em vez de expor a porta:
>
> ```bash
> ssh -L 8000:localhost:8000 usuario@maquina
> ```
>
> Bindar em `0.0.0.0` estêÂ¡ **bloqueado** porque o `textual-serve` embute o host de bind no websocket e o navegador rejeita conexêÂ£o WS pra `0.0.0.0`. êÅ¡til pra:

- Acompanhar de outra mêÂ¡quina na rede local (acesse `http://<ip-da-maquina>:8000`)
- Manter o monitor visêÂ­vel num browser sem ocupar terminal
- Compartilhar visêÂ£o temporêÂ¡ria com colegas na mesma rede

### Ajuda

```bash
./start.sh -h    # mostra os modos disponêÂ­veis
```

### Atalhos de teclado

| Tecla | AêÂ§êÂ£o |
|-------|------|
| `r`   | Refresh manual |
| `a`   | Alterna entre **SessêÂµes ativas** ê¢â â **todas** (vivas + mortas) |
| `p`   | Liga/desliga o painel **Processos** (top N da mêÂ¡quina) |
| `v`   | Liga/desliga o painel **ServiêÂ§os** (Docker + systemd boot) |
| `s`   | Quando Processos visêÂ­vel: alterna ordenaêÂ§êÂ£o **CPU% ê¢â â RAM%** |
| `,`   | Encolhe a coluna esquerda em 5% (chat fica maior) |
| `.`   | Aumenta a coluna esquerda em 5% (chat fica menor) |
| `=`   | Reseta a divisêÂ£o em 50/50 |
| `1`   | Abre **Cota** em fullscreen modal |
| `2`   | Abre **MêÂ¡quina** em fullscreen |
| `3`   | Abre **Processos** em fullscreen |
| `4`   | Abre **SessêÂµes** em fullscreen |
| `5`   | Abre **ServiêÂ§os** em fullscreen (Docker completo + systemd boot completo) |
| **click** no têÂ­tulo de uma seêÂ§êÂ£o | Mesmo efeito do `1`-`4` correspondente |
| `Esc` ou `q` (no modal) | Fecha o modal |
| `q`   | Sair |

Auto-refresh a cada **10 segundos** por padrêÂ£o (configurêÂ¡vel em `config.json` via `refresh_interval_seconds`). A coleta (lêÂª transcripts grandes, roda `nvidia-smi`, `docker ps`, etc.) acontece numa **worker thread** ê¢â¬â a TUI nêÂ£o congela e os valores antigos ficam na tela atêÂ© o novo lote chegar. O subtitle do header mostra **"atualizado hêÂ¡ Xs"** ticando a cada segundo.

Chat tambêÂ©m êÂ© assêÂ­ncrono: ao enviar, a mensagem aparece na hora com `aguardando claude...` enquanto a API responde em background. Mandar nova mensagem antes da anterior chegar êÂ© bloqueado com aviso.

---

## PainêÂ©is da TUI

A TUI tem layout em duas colunas (50/50 por padrêÂ£o, redimensionêÂ¡vel com `,` `.` `=`):

- **Esquerda** ê¢â¬â empilha verticalmente: Cota ê¢â â MêÂ¡quina ê¢â â Processos (opcional) ê¢â â SessêÂµes
- **Direita** ê¢â¬â Chat ocupando toda a altura

Limites de redimensionamento: 20% / 80%. NêÂ£o fica tudo de um lado sêÂ³.

### 1. Cota (topo)

Barra horizontal por janela:

| Linha | Significado |
|-------|-------------|
| `5h`  | Janela rolante de 5 horas ê¢â¬â todos os modelos |
| `7d`  | Janela semanal ê¢â¬â todos os modelos |
| `7dê¢â¢Âª` | Janela semanal **Sonnet** ê¢â¬â limite separado, importante monitorar |

Cor da barra segue os thresholds de `quota_pct` em `config.json`. Se a barra aparecer "aguardando API..." êÂ© porque o cache `/tmp/cc_limits_<uid>.json` estêÂ¡ vazio ou velho (> 2 min).

### 2. MêÂ¡quina

Uso de hardware do servidor ê¢â¬â barras coloridas pelos thresholds de `system_pct`:

| Linha | Mostra |
|-------|--------|
| `CPU`  | `usage_pct` (load1 / cores) + load 1m/5m/15m + nêÂº de cores |
| `RAM`  | `used / total MB` + swap usado |
| `Disk` | `used / total GB` + livre |
| `GPU`  | `nvidia-smi`: utilizaêÂ§êÂ£o %, VRAM, temperatura. Linha some se nêÂ£o houver GPU NVIDIA. |

CPU usa `os.getloadavg()` (nêÂ£o `psutil`), RAM lêÂª `/proc/meminfo`, GPU êÂ© opcional via `nvidia-smi`. Sem dependêÂªncias extras.

### 3. Processos (opcional ê¢â¬â tecla `p`)

Top N processos da mêÂ¡quina, ordenados por **%CPU** ou **%RAM** (alterna com `s`). Default: oculto.

| Coluna | Mostra |
|--------|--------|
| `PID` | identificador do processo |
| `User` | dono (truncado em 10 chars) |
| `CPU%` | % de CPU (cor pelos thresholds de `system_pct`) |
| `RAM%` | % de RAM (mesma escala de cor) |
| `Tempo` | etime (`HH:MM:SS` ou `D-HH:MM:SS`) |
| `Comando` | comm truncado em 40 chars |

Coleta via `ps -eo pid,user,pcpu,pmem,etime,comm --sort=-%cpu` (sem dependêÂªncia extra). êÅ¡til pra:

- Saber **quais sessêÂµes `claude` estêÂ£o consumindo CPU/RAM** (cada PID aparece separado)
- Detectar processos pesados quando o painel MêÂ¡quina ficar ê°Å¸Å¸Â¡/ê°Å¸âÂ´
- Ver `ollama`, GPU workers, dev servers etc. na mesma tela

ConfigurêÂ¡vel: `top_processes_n` no `config.json` (padrêÂ£o `10`). O painel sêÂ³ polla `ps` quando estêÂ¡ visêÂ­vel ê¢â¬â nêÂ£o tem custo quando oculto.

### 4. ServiêÂ§os (opcional ê¢â¬â tecla `v`)

Resumo de **Docker** e **systemd boot** num painel sêÂ³. Default: oculto.

**Inline mostra sêÂ³ o que importa:**

```
Docker   10/44 containers ativos
  ê¢Åâ imkt4-postgres-1     Up 4 days     0.0.0.0:5432->5432/tcp
  ê¢Åâ openclaw-ollama      Up 4 days     0.0.0.0:11434->11434/tcp
  ê¢Åâ open-webui           Up 4 days     0.0.0.0:3000->8080/tcp
  ... +7 ativos êÂ· use 5 pra ver tudo (incl. parados)
Boot     33/102 systemd .service habilitados rodando  (abra modal pra ver lista)
```

**Modal (`5`)** mostra a lista completa: todos os containers (incluindo parados, com cêÂ³digo de saêÂ­da) e todas as services systemd habilitadas com estado atual (`running`, `dead`, `exited`, `failed`).

Coleta:
- Docker via `docker ps -a` ê¢â¬â se `docker` nêÂ£o estiver no `$PATH`, mostra "indisponêÂ­vel"
- Boot via 2 chamadas a `systemctl` (1êâ `list-unit-files --state=enabled`, 1êâ `list-units --all` para o estado) ê¢â¬â sem polling unitêÂ¡rio, leve
- SêÂ³ roda quando o painel estêÂ¡ visêÂ­vel (zero custo quando oculto)

### 5. SessêÂµes (ativas ou todas)

Tabela com uma linha por sessêÂ£o Claude Code. Por padrêÂ£o mostra sêÂ³ sessêÂµes com PID alive; pressione **`a`** para incluir as mortas (sessêÂµes cujo arquivo em `~/.claude/sessions/*.json` ainda existe mas o processo terminou). Mortas aparecem em cinza com status `morta`. Colunas:

| Coluna | DescriêÂ§êÂ£o |
|--------|-----------|
| **Projeto** | Nome do diretêÂ³rio de trabalho (truncado em 20 chars) |
| **Status** | `idle` (verde) ou `busy` (amarelo) |
| **Modelo** | `opus`, `sonnet`, `haiku` ou outro |
| **Ctx%** | % da janela de 1M tokens em uso. Soma `input + cache_read + cache_creation + output` |
| **CLAUDE.md** | Tamanho do `CLAUDE.md` no `cwd` da sessêÂ£o |
| **Mem** | `<n>f/<x>k` ê¢â¬â nêÂºmero de arquivos e KB totais em `memory/` |
| **Agts** | Quantidade de chamadas êÂ  tool `Agent` no transcript |
| **Skls** | Skills distintas invocadas (tool `Skill`) |
| **Update** | Tempo desde o êÂºltimo update (`12s`, `3m`, `1h`) |

Cada mêÂ©trica recebe cor azul/verde/amarelo/vermelho conforme thresholds em `config.json`.

### 6. Chat (lateral)

Chat embutido com Claude com **dois modos**:

**Modo geral** (padrêÂ£o) ê¢â¬â recebe o estado completo do painel como system prompt:

- "Qual sessêÂ£o têÂ¡ usando mais contexto?"
- "Quanto falta pra cota Sonnet?"
- "Algum projeto com memêÂ³ria crêÂ­tica?"

**Modo focado** ê¢â¬â selecione uma linha da tabela (ê¢â â/ê¢â â + **Enter**) e o chat carrega o transcript daquela sessêÂ£o como contexto. AêÂ­ dêÂ¡ pra perguntar sobre o trabalho especêÂ­fico:

- "O que essa sessêÂ£o têÂ¡ fazendo agora?"
- "Qual foi o êÂºltimo erro?"
- "Resume o que foi feito atêÂ© agora."
- "Por que ele tomou essa decisêÂ£o?"

> O chat êÂ© um **observador analêÂ­tico** ê¢â¬â nêÂ£o interage com o agente daquela sessêÂ£o, sêÂ³ lêÂª o transcript. Pra interaêÂ§êÂ£o remota real, ver V2 no roadmap.

#### Comandos do chat

| Comando | AêÂ§êÂ£o |
|---------|------|
| `/help`   | Ajuda completa (TUI + chat + skills + roadmap) |
| `/diag`   | DiagnêÂ³stico do painel ê¢â¬â lista cota e sessêÂµes em alerta (ê°Å¸Å¸Â¡/ê°Å¸âÂ´) com sugestêÂµes |
| `/skills` | 3 skills êÂºteis pra rodar nas sessêÂµes focadas (statusline / memory-audit / handoff) |
| `/clear`  | Volta ao modo geral (limpa o foco e o histêÂ³rico) |
| `/log`    | Mostra as êÂºltimas 20 linhas do log de chat |
| `/where`  | Imprime os caminhos do log, config e diretêÂ³rio de projects |
| `/docker` | sem args: lista containers rodando êÂ· `/docker <nome> start\|stop\|restart\|logs` age direto |
| `/fork`   | *(roadmap V2)* ê¢â¬â abrir nova sessêÂ£o Claude Code continuando a focada |

Trocar de foco tambêÂ©m limpa o histêÂ³rico do chat ê¢â¬â pra nêÂ£o misturar conversas.

#### DiagnêÂ³stico do painel ê¢â¬â `/diag`

Roda regras locais sobre os thresholds do `config.json` e devolve sêÂ³ o que estêÂ¡ em **ê°Å¸Å¸Â¡ yellow** ou **ê°Å¸âÂ´ red**:

- Cotas (5h, 7d, 7d Sonnet) em alerta
- SessêÂµes com `Ctx%` alto (risco de auto-compaction)
- SessêÂµes com memêÂ³ria ou `CLAUDE.md` inchados
- SessêÂµes com muitos agentes lanêÂ§ados

Quando algo aparece, sugere a skill apropriada do `skillmanager3x` (`session-handoff`, `memory-audit`, etc).

#### Skills êÂºteis ê¢â¬â `/skills`

Lista as 3 skills do [skillmanager3x](https://github.com/linnoff88-dev/skillmanager3x) com **quando usar cada uma**:

| Skill | Quando rodar |
|-------|--------------|
| **session-statusline** | Checkpoint operacional rêÂ¡pido durante o trabalho. Triggers: "onde estamos", "checkpoint", "organiza a sessêÂ£o". |
| **memory-audit**       | Quando o painel mostra memêÂ³ria ê°Å¸Å¸Â¡/ê°Å¸âÂ´ ou CLAUDE.md grande. Triggers: "analise a memêÂ³ria", "memêÂ³ria inchada", "limpa memêÂ³ria". |
| **session-handoff**    | Antes de `/clear` ou quando ctx ê°Å¸âÂ´. Gera resumo pra um agente fresco continuar. Triggers: "vou dar /clear", "handoff", "encerrar sessêÂ£o". |

Como rodar: na sessêÂ£o Claude Code alvo, digite o trigger ou `/<skill-name>`.

#### Log de chat

Cada turno (vocêÂª ê¢â â claude) e cada erro sêÂ£o gravados em `chat.log` no diretêÂ³rio do projeto. Inclui:

- timestamp, role (user/claude), foco (`[geral]` ou `[focus=<8 chars>]`), e texto
- em caso de erro: status code e body cru da resposta da API

êÅ¡til pra debugar 401 (OAuth expirado/invêÂ¡lido), rate limit, payload invêÂ¡lido, etc. O arquivo estêÂ¡ no `.gitignore`.

Modelo padrêÂ£o: `claude-haiku-4-5-20251001` (configurêÂ¡vel em `config.json` ê¢â â `chat.model`). AutenticaêÂ§êÂ£o via OAuth do Claude Code (token em `~/.claude/.credentials.json`).

---

## Fontes de dados

| MêÂ©trica | Fonte |
|---|---|
| SessêÂµes ativas | `~/.claude/sessions/*.json` ê¢â¬â um arquivo por PID, filtrado por processo vivo (`os.kill(pid, 0)`) |
| Modelo, contexto %, custo | êÅ¡ltima mensagem `assistant` com `usage` no transcript JSONL **ativo** em `~/.claude/projects/<encoded-cwd>/`. O `sessionId` em `session.json` congela no boot ê¢â¬â apêÂ³s `/clear` o Claude Code cria um JSONL novo e o monitor segue o de `mtime` mais recente |
| Cota 5h / 7d / 7d-Sonnet | `/tmp/cc_limits_<uid>.json` ê¢â¬â cache do statusline (fonte oficial via API Anthropic) |
| `CLAUDE.md` | `stat` do arquivo em `<cwd>/CLAUDE.md` |
| MemêÂ³ria | Conta arquivos e soma bytes de `~/.claude/projects/<encoded-cwd>/memory/` |
| Agentes lanêÂ§ados | Tool uses `name == "Agent"` no transcript (acumulado na sessêÂ£o) |
| Skills invocadas | Tool uses `name == "Skill"` no transcript (distintas) |
| Chat OAuth | `~/.claude/.credentials.json` ê¢â â `claudeAiOauth.accessToken`. Header: `anthropic-beta: oauth-2025-04-20` |

> A cota **nêÂ£o** êÂ© recalculada localmente como `ccm`/`ccusage` fazem ê¢â¬â êÂ© lida do cache que o prêÂ³prio Claude Code mantêÂ©m com os valores oficiais da API. Isso evita divergêÂªncia.

### Encoding do `cwd`

Barras e dois pontos viram hêÂ­fens, com hêÂ­fen prefixado:

```
/home/user/projetos/foo  ê¢â â  -home-user-projetos-foo
```

O monitor tenta ambas as variaêÂ§êÂµes (com e sem hêÂ­fen prefixado) ao localizar transcript e diretêÂ³rio de memêÂ³ria.

---

## ConfiguraêÂ§êÂ£o ê¢â¬â `config.json`

EditêÂ¡vel sem tocar no cêÂ³digo. Estrutura:

### `thresholds`

Limites de cor por mêÂ©trica. Cada mêÂ©trica tem 4 nêÂ­veis: `blue` ê¢â°Â¤ `green` ê¢â°Â¤ `yellow` ê¢â°Â¤ `red`.

| MêÂ©trica | O que mede | Defaults (b/g/y/r) |
|---------|------------|--------------------|
| `quota_pct`           | % da cota usada (5h, 7d, 7d-Sonnet) | 20 / 60 / 85 / 95 |
| `context_pct`         | % da janela de contexto da sessêÂ£o. >90% = risco de auto-compaction | 10 / 50 / 75 / 90 |
| `memory_kb`           | Tamanho total da memêÂ³ria do projeto (KB). >600 = considere `/memory-audit` | 0 / 100 / 300 / 600 |
| `memory_files`        | NêÂºmero de arquivos de memêÂ³ria. >35 = fragmentaêÂ§êÂ£o | 0 / 10 / 20 / 35 |
| `claude_md_bytes`     | Tamanho do `CLAUDE.md`. >40KB = pode impactar contexto | 0 / 8192 / 20480 / 40960 |
| `agents_per_session`  | Agentes lanêÂ§ados na sessêÂ£o. >30 = sessêÂ£o muito pesada | 0 / 5 / 15 / 30 |
| `cost_per_session_usd`| Custo acumulado em USD. >$20 = revisar eficiêÂªncia | 0 / 2 / 8 / 20 |

Esquema visual:

| Cor | Significado |
|-----|-------------|
| ê°Å¸âÂµ Azul | Muito baixo / ocioso |
| ê°Å¸Å¸Â¢ Verde | Normal / saudêÂ¡vel |
| ê°Å¸Å¸Â¡ Amarelo | AtenêÂ§êÂ£o |
| ê°Å¸âÂ´ Vermelho | CrêÂ­tico / alerta |

### `alerts`

Habilita/desabilita mensagens de alerta individualmente:

| Chave | PadrêÂ£o | Mensagem |
|-------|--------|----------|
| `quota_5h_red`  | enabled  | ê¢Å¡Â  Cota 5h crêÂ­tica! |
| `quota_7d_red`  | enabled  | ê¢Å¡Â  Cota semanal crêÂ­tica! |
| `context_red`   | enabled  | ê¢Å¡Â  Contexto alto ê¢â¬â risco de compaction |
| `memory_red`    | enabled  | ê¢Å¡Â  MemêÂ³ria excessiva ê¢â¬â rode `/memory-audit` |
| `claude_md_red` | disabled | ê¢Å¡Â  CLAUDE.md muito grande |
| `cost_red`      | enabled  | ê¢Å¡Â  SessêÂ£o cara |

### `chat`

| Campo | PadrêÂ£o | DescriêÂ§êÂ£o |
|-------|--------|-----------|
| `model`      | `claude-haiku-4-5-20251001` | Modelo do chat embutido |
| `max_tokens` | `1024` | Tamanho mêÂ¡ximo da resposta |

### Outros

| Campo | PadrêÂ£o | DescriêÂ§êÂ£o |
|-------|--------|-----------|
| `title` | `ARCKHÉ Claude Monitor` | TêÂ­tulo do header (a versêÂ£o êÂ© appendada automaticamente) |
| `refresh_interval_seconds` | `10` | Intervalo de auto-refresh em segundos |

---

## Arquitetura

```
iccmonit/
ê¢âÅê¢ââ¬ê¢ââ¬ monitor.py     # TUI principal (Textual) ê¢â¬â V1
ê¢âÅê¢ââ¬ê¢ââ¬ serve.py       # Wrapper de modo web via textual-serve
ê¢âÅê¢ââ¬ê¢ââ¬ start.sh       # Entrypoint: terminal e modo web
ê¢âÅê¢ââ¬ê¢ââ¬ config.json    # Thresholds, alertas, chat, refresh
ê¢âÅê¢ââ¬ê¢ââ¬ README.md      # Este arquivo
ê¢âÅê¢ââ¬ê¢ââ¬ CLAUDE.md      # InstruêÂ§êÂµes do projeto pra Claude Code
ê¢âÅê¢ââ¬ê¢ââ¬ .gitignore
ê¢ââê¢ââ¬ê¢ââ¬ docs/
    ê¢âÅê¢ââ¬ê¢ââ¬ img/         # Screenshots
    ê¢âÅê¢ââ¬ê¢ââ¬ PLAN.md      # Plano original e decisêÂµes de arquitetura
    ê¢ââê¢ââ¬ê¢ââ¬ RESEARCH.md  # Pesquisa das fontes de dados do Claude Code
```

### Componentes do `monitor.py`

| SêÂ­mbolo | Papel |
|---------|-------|
| `load_sessions()` | LêÂª `~/.claude/sessions/*.json` e filtra por PID vivo |
| `get_transcript_usage()` | LêÂª êÂºltima `assistant.usage` do transcript via `tac` |
| `get_session_extras()` | Calcula tamanho de `CLAUDE.md`, memêÂ³ria, contagem de agentes/skills |
| `load_quota()` | LêÂª `/tmp/cc_limits_<uid>.json` (descarta se > 120s) |
| `get_oauth_token()` | Extrai `accessToken` do `.credentials.json` |
| `QuotaBar` | Widget com barras de cota |
| `SessionTable` | DataTable com sessêÂµes |
| `ChatPane` | RichLog + Input ê¢â¬â chat com `system` montado a cada mensagem com snapshot do painel |
| `MonitorApp` | App Textual; auto-refresh via `set_interval` |

---

## Versionamento

Constante `VERSION` em `monitor.py` no formato **`v1.xx.yy`**:

- **major (`1`)** ê¢â¬â sêÂ³ muda na transiêÂ§êÂ£o para V2
- **`xx`** ê¢â¬â incrementa a cada **recurso novo** (sequencial: `01`, `02`, ...)
- **`yy`** ê¢â¬â incrementa a cada **bug fix** (sequencial dentro da major)
- **`yy` NêÆO reinicia** quando `xx` sobe ê¢â¬â sêÂ³ zera junto com `xx` na transiêÂ§êÂ£o de major

Exemplos:
- `v1.00.00` ê¢â â estado inicial
- `v1.01.00` ê¢â â primeiro recurso novo (nenhum bug fix ainda)
- `v1.01.02` ê¢â â apêÂ³s dois bugs fixados dentro do recurso `01`
- `v1.02.02` ê¢â â prêÂ³ximo recurso (`yy` mantêÂ©m em `02` porque nenhum novo bug foi corrigido)
- `v1.02.03` ê¢â â bug fix sobre `v1.02.02`
- `v2.00.00` ê¢â â mudanêÂ§a de major zera tudo

A versêÂ£o êÂ© exibida no têÂ­tulo da janela TUI (header).

---

## Roadmap

- **V1** *(atual)* ê¢â¬â painel de sessêÂµes + cota + chat geral + chat focado em sessêÂ£o (read-only) + modo web
- **V2** *(planejado)* ê¢â¬â interaêÂ§êÂ£o remota com sessêÂ£o ativa: injetar prompts numa sessêÂ£o como se estivesse no terminal original. Mecanismo a investigar:
  - stdin do processo Claude Code
  - arquivo de IPC
  - ou `claude --resume <sessionId>` num processo paralelo (forka, nêÂ£o dirige a sessêÂ£o original)
  - sem API oficial ê¢â¬â requer pesquisa

---

## SoluêÂ§êÂ£o de problemas

| Sintoma | Causa provêÂ¡vel | Como resolver |
|---------|----------------|---------------|
| Cota mostra `aguardando API...` | `/tmp/cc_limits_<uid>.json` ausente ou > 2 min | Confirme que o statusline padrêÂ£o do Claude Code estêÂ¡ rodando ê¢â¬â ele popula esse cache |
| Chat indisponêÂ­vel | Token OAuth nêÂ£o encontrado | FaêÂ§a login novamente no Claude Code e confirme que `~/.claude/.credentials.json` tem `claudeAiOauth.accessToken` |
| Chat retorna `401` | Token OAuth expirado, revogado, ou cota da janela esgotada | Use `/log` no chat pra ver o body do erro. Se expirado, faêÂ§a `claude logout` + `claude login`. Se cota, espere a janela renovar (vide painel de cota). |
| Tabela vazia | Nenhuma sessêÂ£o Claude Code ativa | Abra uma sessêÂ£o `claude` em outro terminal e dêÂª `r` para refresh |
| `Ctx%` zerado | Transcript ainda sem mensagem `assistant` com `usage` | Use a sessêÂ£o um pouco ê¢â¬â depois da primeira resposta o valor aparece |
| Modo web nêÂ£o abre | Porta ocupada | Tente outra porta: `./start.sh web 9001` |
| Modo web abre o "demo" do Textual | VersêÂ£o antiga sem `textual-serve` | Atualize: `git pull` (a partir de v1.01.01 usa `textual-serve` separado) |
| Modo web mostra sêÂ³ o logo "iccmonit" e nêÂ£o carrega a TUI | VersêÂ£o `v1.01.01` bindando em `0.0.0.0` (websocket invêÂ¡lido no navegador) | Atualize pra `v1.01.02`+ ê¢â¬â bind virou `127.0.0.1` |
| ApêÂ³s `/clear` o `Ctx%` continua alto | VersêÂ£o ê¢â°Â¤ `v1.02.00` lia o JSONL antigo (sessionId congelado em `session.json`) | Atualize pra `v1.02.01`+ ê¢â¬â passa a seguir o JSONL mais recente (`mtime`) |
| Erro `ImportError: textual` | DependêÂªncias nêÂ£o instaladas | Rode `pip install textual anthropic --break-system-packages` ou simplesmente `./start.sh` (instala automaticamente) |

---

## Por que nêÂ£o usar X?

- **`ccm` / claude-monitor** ê¢â¬â calcula cota a partir dos JSONL locais e diverge da cota real da API. Aqui usamos o cache oficial.
- **`ccusage`** ê¢â¬â ferramenta de auditoria/relatêÂ³rio, nêÂ£o tempo real.
- **`claude-pulse`** ê¢â¬â o `statusline-command.sh` existente jêÂ¡ cobre o mesmo escopo.

---

## LicenêÂ§a

MIT.

---

Projeto  - ARCKHÉe
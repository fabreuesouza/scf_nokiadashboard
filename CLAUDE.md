# Contexto para o Claude — continuação entre sessões

Leia isto antes de qualquer tarefa. Este arquivo é carregado automaticamente em novas sessões.

## O projeto

- Ferramenta web **de arquivo único**: `NokiaSCF_ReaderDashboard_v2.08.html` (~8.8k linhas, HTML + CSS + JS inline, sem build, sem dependências além de Google Fonts).
- Lê arquivos SCF (XML de configuração de sites Nokia LTE/NR) no navegador e gera módulos de análise. Versão exibida no header: **v2.14** (o nome do arquivo continua `_v2.08.html`).
- O dono trabalha com engenharia/otimização de redes Nokia. Comunicação em **português**; código, comentários e commits em **inglês**.

## Fluxo de trabalho combinado (fazer sempre, sem perguntar)

1. Editar o HTML.
2. Validar a sintaxe do JS: extrair os blocos `<script>` para um arquivo no scratchpad e rodar `node --check`.
3. Commit (mensagem em inglês, curta) e `git push -u origin <branch da sessão>`. O GitHub às vezes devolve `Internal Server Error` no push: repetir com backoff (2s, 4s, 8s, 16s).
4. Republicar o artifact **no mesmo URL**: `Artifact publish` com `file_path` do HTML e `url: https://claude.ai/artifact/AvioGaGJ1P6afpMdN1VnaD`.
5. Responder em português, curto, dizendo o que mudou e que não foi testado com SCF real (quando for o caso).

- O aviso do artifact sobre "download link" é conhecido e pode ser ignorado (botões de exportar não funcionam dentro do viewer).
- **A cada update publicado, incrementar a versão no header** (`<span class="logo-badge" ...>vX.YY</span>` no `<h1>`): +0.01 por update (v2.09 → v2.10 → …) e atualizar a versão citada neste arquivo. O nome do arquivo não muda.
- Não criar PR sem pedido explícito.
- Se o nome do arquivo mudar de versão, atualizar este CLAUDE.md e o README.md.

## Design system (Blue Design)

- Visual vem do `blue-design.css` do repo `fabreuesouza/design-system-css`, branch `claude/blue-design` (ainda não está na `main`), com os tokens (`:root` dos temas) **embutidos** no `<style id="blue-design-tokens">` do `<head>`, copiados do commit `5e3a24b`. O viewer do artifact bloqueia CSS externo, então não usar `<link>` para o jsDelivr; ao atualizar o design system, recopiar o bloco TOKENS do `blue-design.css`.
- Logo após os blocos de tokens próprios da ferramenta existe um bloco `:root` de mapeamento (`--bg-body: var(--bg)`, `--brand-blue: var(--accent)`, tech 2G/3G/4G/5G → `--num/--den/--kpi/--feature`, etc.). Para mudar cores, mexer nesse mapeamento, não nos hex antigos.
- Tema escuro é o padrão; claro segue `prefers-color-scheme: light` ou `<html data-theme="light">`. Header sempre escuro, com o filete azul→ciano→âmbar.

## Base de parâmetros Nokia (`fabreuesouza/nokia_database`)

- Repositório privado com tabelas MIND (CSV) de parâmetros/contadores/KPIs. Usado para definições, fórmulas de conversão (`internal_value_formula`) e textos explicativos dos checks.
- Em sessão nova: anexar com `add_repo` (owner `fabreuesouza`, repo `nokia_database`), clonar em `/home/user/nokia_database` e chamar `register_repo_root`.
- Arquivos: `data/parameters/{lte,nr,wcdma,gsm}-mind-parameters.csv`. Ler com Python `csv` (`python3 -I`), filtrando por `parameter_id` e `moc`.
- A ferramenta **não** consulta a base em runtime: textos e fórmulas são embutidos no HTML.

## Mapa do código (números de linha aproximados)

| Item | Onde |
|---|---|
| CSS dos modais / tabela de detalhe | `.hc-detail-table` ~529, `#healthDetail*` ~539, `.modal-content` ~625 |
| Estado global (`scfData`, `lncelFddParams`, `nrcellRfParams`) | ~1473 |
| Helpers: `getParamCaseInsensitive`, `getMOsByClass`, `getParentDistName` | ~1561–1610 |
| `renderHcDetailTable(cols, rows)` — tabela HTML do detalhe dos cards | ~2909 |
| Funções `check*` do Health Check | ~1600–3750 |
| `HEALTH_CHECKS` (registro de cards: id, category, icon, title, run) | ~3754 |
| `openHealthDetailModal` | ~3936 |
| NR Anchoring (`renderNrAnchoringDashboard`) | ~3984 |
| `switchMainTab` (navegação entre módulos) | ~4218 |
| Parser do SCF (monta `scfData`, `lncelFddParams`, objetos de célula) | ~4540–4700 |
| Neighbor Audit / Mapping (`renderNeighborMapping`, `renderNeighborMappingGraph`) | ~5185 / ~5989 |
| Power & Footprint (`renderPowerFootprintDashboard`) | ~7600 |

Use `grep -n` pelo nome da função em vez de confiar nos números.

## Convenções dos checks do Health Check

- Cada `check*(ctx)` retorna `{ status: 'OK'|'WARN'|'FAIL'|'NA', detail, fullDetail }`.
- `fullDetail` é HTML renderizado em área com `white-space: pre-wrap`: `\n` vira quebra de linha. O espaço entre tags é removido ao abrir o modal (`.replace(/>\s+</g,'><')`), então separar blocos com texto/`\n`, não com indentação.
- Tabelas de detalhe: `renderHcDetailTable([{ key, label, mono }], rows)`; células aceitam HTML (ex.: `<span>` vermelho). Colunas `icon`/`status` são coloridas automaticamente.
- Ícones nas linhas: ✅ OK, ⚠️ WARN, ❌ FAIL, ➖ não encontrado.
- Mostrar valor convertido e o raw entre parênteses, ex.: `-110dBm(46)`, `1.5dB(15)`.

## Conversões confirmadas

| Parâmetro | Conversão |
|---|---|
| `b1ThresholdNrRsrp` (ENDCDMEASCONF) | dBm = raw − 156 |
| `a2ThresholdReleaseSsbRsrp` (NRCELL) | dBm = raw − 156 (TS 38.133) |
| `threshold1`, `threshold2a`, `threshold2InterFreq`, `threshold3(a)InterFreq` | dBm = raw − 140 |
| `a3Offset`, `hysA3Offset`, `hysThreshold3` (LNCEL) | dB = raw / 2 |
| `b2Threshold2UtraRscp` (LNHOW) | dBm = raw − 115 |
| `b2Threshold2UtraEcn0` (LNHOW) | dB = raw / 2 − 24 |
| `threshold2Wcdma`, `threshold2GERAN`, `threshold3`, `threshold3a` (LNCEL) | dBm = raw − 140 |
| `pMax` | dBm = raw / 10 |
| `dlCellPwrRed` | dB = raw / 10 |
| `dlpcMimoComp` (LNCEL_FDD) | dB = raw / 100 |
| `dlRsBoost` | enum (`dB3`, `dB1_77`…) → lido com `hcDbEnum` |
| pB (derivado) | `getEffectivePbIndex(boost, allowPbIndexZero)`; B no_RS = A + 10·log10(ρB/ρA) via `parsePbToDb(allowPbIndexZero, dlRsBoost)` |
| `b2Threshold2RssiGERAN` | dBm = raw − 110 |

## Regras de negócio já decididas

- **5G NSA LEG**: OK só se Δ = b1 − a2 estiver entre **+3 e +8 dB**; fora disso FAIL. Detalhe tem glossário dos dois parâmetros.
- **NR Anchoring**: cards unlocked primeiro, depois locked, ordenados por número do gNB.
- **CSFB GERAN Target Cell Identity (LNADJG)**:
  - LNRELG ↔ LNADJG associados por mesma LNBTS + `ci`/`lac` (LNRELG) = `gTargetCi`/`gTargetLac` (LNADJG).
  - Colunas: ícone, MO, Related LNCEL/LNRELG, CI, LAC, RAC, ARFCN, Band, BSIC, Status. (Colunas `targetCellDn`/`targetName` foram testadas e removidas — sem dado no SCF.)
  - ARFCN do LNADJG deve estar no `redirGeranArfcnStructL` dos REDRT GSM de cada LNCEL relacionado; ausente → vermelho + FAIL.
  - Par ARFCN + BSIC duplicado na mesma LNBTS → FAIL (tabela própria).
- **Cell → RMOD** (`buildLcellRmodMap`, `getCellRmods`): LTE `LNCEL.lcrId` → `MNL/.../CELLMAPPING/LCELL-<lcrId>/CHANNELGROUP/CHANNEL.antlDN` → `RMOD-x/ANTL-y` (exibe `RMOD-3 (ANTL-1/2)`); NR `NRCELL.lcrId` → `LCELNR-<lcrId>/.../CHANNEL.resourceDN` → `RMOD-x/PHYANT-y` (exibe `RMOD-21 (PHYANT-1, 8T4R)`, contando canais TX/RX).
- **Cell → RET** (`buildAntlRetuMap`, `formatCellRet`): `ALD-x/RETU-y.antlDNList` lista as ANTL do RMOD; cruzando com as ANTL da célula obtém-se o tilt. `angle` em 0.1° (110 → 11.0°). NR (PHYANT) normalmente sem RETU → N/A.
- **Power & Footprint**: células NB-IoT (LNCEL com filho NBIOT_FDD) ocultas da tabela e dos filtros (`getPowerVisibleDataset`); filtros laterais por Sector (ANTL-1/2→1, 3/4→2, 5/6→3; só LTE) e RMOD; colunas: Cell, RMOD, RET Tilt, Bandwidth, PMax, PwrRed, DL RS Boost, qrxLevMin (SIB-0), Power (W), Spectral Density, Symbol Level (LTE: 3 linhas — RE Type A, B RS, B no_RS, mesmas fórmulas do Boost Analytics; NR: só A), FSPL (f e d = 0.5 km em cinza na célula).
- **Neighbor Analytics → Audit GSM** (`renderNeighborGsmAudit`, view `neighborGsmView`, tab `neighborGsm`): por LNCEL, lista as LNRELG resolvidas para a LNADJG (mesma regra LNBTS + CI/LAC). Colunas: LNRELG, LNADJG, CI, LAC, RAC, ARFCN, Band, BSIC, eNACC, SRVCC, Redir w/ SI, RIM, DTM, Issues. Issues: LNRELG órfã, ARFCN fora do REDRT GSM do LNCEL, ARFCN+BSIC duplicado. Cabeçalho de colunas repetido sob cada linha de LNCEL (que mostra também o TAC). Tabela extra de LNADJG não usadas. Ordem do menu: Audit, Audit GSM, Mapping. GSM → LTE não existe no SCF (precisa do dump da BSC).
- **Health Check — revisão LTE (v2.11)**:
  - Células LNCEL com `administrativeState=locked` são excluídas de todos os checks (e seus filhos); cada card lista as locked com 🔒 / N/A e há um chip no topo. Card próprio "Locked / Barred Cells".
  - A3: margem efetiva = a3Offset + hysA3Offset (dB); WARN fora de 1–6 dB.
  - dlRsBoost: boost ≠ 0 continua WARN; o pB é derivado (36.213 Tab. 5.2-1, 2/4 Tx): 0dB→pB0 (pB1 se `allowPbIndexZero=false`, WARN legado), 3dB→pB1, 4.77dB→pB2, 6dB→pB3; qualquer outro boost (ou negativo) → FAIL (Critical).
  - A2 Stop/Start Margin: threshold2a ≥ max(threshold2InterFreq, threshold2Wcdma se há LNRELW, threshold2GERAN se há LNRELG) + 3 dB, senão FAIL.
  - B2 UTRA: WARN se b2Threshold1Utra > threshold2Wcdma do LNCEL.
  - qrxlevmin: mostra piso efetivo (qrxlevmin + qRxLevMinOffset); WARN se > −110 dBm ou > 6 dB da mediana da mesma EARFCN.
  - VoLTE t310/n310: WARN se t310Qci1/n310Qci1 < t310/n310 do SIB.
  - Novos cards: Idle↔Connected Inter-Freq Trigger (|Δ| > 6 dB WARN), A5 Inter-Freq vs A2 Activation (FAIL se threshold3InterFreq > threshold2InterFreq), Intra-Freq A5 (threshold3a − threshold3 fora de 0–6 dB WARN), CIO Range & Asymmetry (|CIO| > 6 dB ou par co-site assimétrico WARN), Reselection Priority Map (mesma EARFCN com prioridades diferentes SIB/IRFIM → FAIL), TAC Consistency per Site, PCI Mod-3/Mod-30 co-site (mesmo PCI ou mod 3 → FAIL, mod 30 → WARN), PRACH Root Sequence Overlap (faixas sobrepostas → FAIL, Ncs pela Tab. 5.7.2-2), Locked/Barred Cells.
  - Helpers compartilhados: `hcTtt`, `hcDbEnum`, `hcCellEarfcn`, `hcSibOf`, `hcBtsOf`, `hcLncelOf`.
- **Guidance nos cards** (`HC_GUIDANCE`, `addHcGuidance`): quando o status é WARN/FAIL, o detalhe ganha um bloco âmbar logo após o resumo com "Why it matters" e "Ideal setup (RF good practice)" com valores típicos (dBm(raw)). Card novo → adicionar entrada no `HC_GUIDANCE`.
- **EARFCN junto do LNCEL**: toda tabela do detalhe do Health Check que lista LNCEL ganha a coluna EARFCN logo após a coluna da célula (`hcAddEarfcnColumns`, aplicado no modal; tabelas que já têm EARFCN ficam como estão).
- **Modal do Health Check**: 1080px de largura; fonte do detalhe `calc(0.76rem - 1px)`, tabela `calc(0.7rem - 1px)`.

## Agentes

- `lte-agent` (`.claude/agents/lte-agent.md`): especialista LTE Nokia; analisa e recomenda melhorias em toda a parte LTE do viewer e escreve prompts prontos para busca de dados no SCF e organização na ferramenta, usando o `nokia_database` e 3GPP como fonte. Só edita código se pedido.

## Pendências / ideias

- Ampliar o uso do `nokia_database` com glossários em outros cards (mesmo padrão do 5G NSA LEG).
- Exportações/downloads não funcionam dentro do artifact viewer (precisariam da capability `downloads`).

# Nokia SCF Analytics & Hardware Platform

Ferramenta web de arquivo único (`NokiaSCF_ReaderDashboard_v2.08.html`) para ler e auditar arquivos SCF (Site Configuration File) de sites Nokia LTE/NR. Roda inteiramente no navegador: carregue o SCF e os módulos são calculados localmente, sem backend.

Versão atual: **v2.08** (exibida no header da ferramenta).

## Como usar

- Abra `NokiaSCF_ReaderDashboard_v2.08.html` direto no navegador, ou
- Sirva a pasta localmente: `python -m http.server 8743` (configuração em `launch.json`; `serve.ps1` para Windows) e acesse `http://localhost:8743/NokiaSCF_ReaderDashboard_v2.08.html`.

## Módulos

| Módulo | O que faz |
|---|---|
| SCF Analytics | Visão geral do SCF, contagem de MOs por tecnologia |
| Boost Analytics | Análise de RS boost / potência de referência |
| CIO Analytics | Cell Individual Offsets das relações de vizinhança |
| Power & Footprint | pMax, dlCellPwrRed, PSD (W/MHz) e potência por RE |
| Neighbor Analytics → Audit | Auditoria de relações LNREL/LNRELW/LNRELGNBCELL |
| Neighbor Analytics → Mapping | Árvore Source → LNBTS → LNCEL com perfil de eventos A1/A2/A3/A5, B1/B2 inter-RAT e detecção de conflitos |
| NR Anchoring | gNBs vizinhos (LNADJGNB) × relações LNRELGNBCELL para EN-DC |
| Hardware Mapping | SMOD/BBMOD/RMOD e diagrama de configuração |
| Site Health Check | Cards de checagem por categoria (Mobility, CS Fallback, Power & Coverage, ...) |
| CM Editor | Edição de parâmetros |

## Base de parâmetros

As definições, fórmulas de conversão e textos explicativos dos checks usam como referência o repositório [`fabreuesouza/nokia_database`](https://github.com/fabreuesouza/nokia_database) (tabelas MIND de parâmetros LTE/NR/WCDMA/GSM). A ferramenta não consulta essa base em tempo de execução: os textos e fórmulas são embutidos no HTML.

## Changelog

### v2.08 (atual)

**Base**
- Consolidação do repositório: o `NokiaSCF_ReaderDashboard_v60.html` (v1.86) e a cópia duplicada `Nokia SCF Analytics & Hardware Platform.html` foram substituídos pelo v2.08, que inclui o módulo Neighbor Mapping, novos checks de CSFB GERAN e a correção de LNBTS (antes MRBTS) na identificação de relações intra-site.

**NR Anchoring**
- Cards ordenados com gNBs unlocked primeiro, depois locked, ambos pelo número do gNB.

**Health Check – 5G NSA LEG**
- Passa apenas quando Δ = `b1ThresholdNrRsrp` − `a2ThresholdReleaseSsbRsrp` está entre **+3 e +8 dB**; fora disso é FAIL.
- Glossário no detalhe explicando os dois parâmetros (B1 = adição do SgNB, A2 = release do SgNB) e o motivo da janela; cada par reprovado indica o lado violado e o risco.
- Conversão de `a2ThresholdReleaseSsbRsrp` corrigida para `dBm = raw − 156` (antes −157), alinhada ao 3GPP TS 38.133.

**Power reduction (`dlCellPwrRed`)**
- Valor dividido por 10 (raw 15 → 1.5 dB) na coluna PwrRed do Power & Footprint, no cálculo de potência por RE e no card de Power Reduction do Health Check.

**Health Check – CSFB GERAN Target Cell Identity (LNADJG)**
- Coluna "Related LNCEL/LNRELG" (2ª coluna): LNRELG associados a cada LNADJG pela mesma LNBTS + `ci`/`lac` = `gTargetCi`/`gTargetLac`.
- ARFCN do LNADJG conferido nas listas `redirGeranArfcnStructL` dos REDRT GSM de cada LNCEL relacionado; ARFCN e LNCEL ausentes marcados em vermelho.
- Nova tabela de pares **ARFCN + BSIC (NCC/BCC) duplicados** entre LNADJG da mesma LNBTS (FAIL).

**Health Check – modal**
- Modal 20% mais largo (900 → 1080 px) e fonte do detalhe 1 px menor.
- Removidos os espaços em branco extras no topo do detalhe e entre títulos e tabelas.

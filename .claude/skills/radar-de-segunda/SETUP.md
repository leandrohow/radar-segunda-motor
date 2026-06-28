# Setup — Radar de Segunda

Quatro passos pra colocar o motor pra rodar. Os passos 1 e 2 são manuais e únicos. O 3 é o agendamento. O 4 é só pra entender como ele encaixa no resto.

**Onde isso roda.** O Radar de Segunda vive na conta de **time da How** (onde a Amanda tem acesso) e opera sozinho ali. É independente do Radar de Sábado, que é skill pessoal e fica na conta do Leandro. Por isso este pacote é auto-contido: não lê arquivo de nenhuma outra skill nem de outra conta. Os recursos que ele usa (pasta no Drive, Painel, ledger) vivem no contexto do time.

---

## 1. Pasta no Drive ✅ (feito)

A skill guarda o ledger numa pasta fixa do Drive. A busca do Drive não acha JSON, então o ID fica cravado na skill — **já está**.

- Pasta: **"Radar de Segunda"** · ID `1fWtiWlGgrsIomxI66EvLUUrRomx1eEEm`
- URL: https://drive.google.com/drive/folders/1fWtiWlGgrsIomxI66EvLUUrRomx1eEEm
- Já configurado em `references/ledger.md` (bloco "Configuração do Drive").

**Importante:** como o motor roda na conta de **time**, essa pasta precisa estar num Drive que o time acessa — um Drive compartilhado, ou pasta compartilhada com a Amanda. Não use a sua pasta pessoal (é onde mora o Radar de Sábado, que é outra conta). Os dois radares ficam em Drives diferentes de propósito. Confira que o link acima está com acesso pra conta de time.

---

## 2. Painel de números (5 min)

A interface manual onde você e a Amanda lançam os números dos posts. Escolha **uma** opção (detalhe em `references/ledger.md`):

- **Opção A — Google Sheet (recomendada).** Crie uma planilha "Painel — Radar de Segunda" com as colunas: `conta`, `plataforma`, `url_post`, `data_post`, `gancho_id`, `alcance`, `impressoes`, `salvamentos`, `comentarios`, `compartilhamentos`, `delta_seguidores`, `nota_perfil`. Uma linha por post, preenchida 5–7 dias depois de publicar.
- **Opção B — Drop no Drive.** Um `numeros-AAAA-Wnn.json` por semana na pasta, padrão append-only do ledger.

Regra de ouro do loop: **se os números não entram, o motor fica cego.** É a única disciplina semanal inegociável — sua e da Amanda. Cinco minutos por semana lançando números é o que faz a coisa aprender em vez de chutar.

Quando plugar a ferramenta (ver fim deste arquivo), ela passa a alimentar esse mesmo Painel e nada mais muda.

---

## 3. Agendamento da routine (motor)

O Radar de Segunda roda como routine agendada no Claude Code, no mesmo molde do motor do Radar de Sábado e do `pipeline-watchdog`.

- **Quando:** começo de semana — domingo à noite ou segunda de manhã, pra a pauta estar na mesa quando a semana abre.
- **O que a routine faz, em ordem:**
  1. Lê `references/fontes.md` (mapa de fontes completo — substância + conversa; autônomo, não lê arquivo de outra skill).
  2. Lê o ledger na pasta do Drive (todos os `ledger-*.json`, união).
  3. Lê o Painel e grava os números novos no ledger.
  4. Varre as três camadas (em paralelo via subagentes no Claude Code; em série no Cowork/chat).
  5. Filtra por perna e novidade (`references/ganchos.md` + ledger).
  6. Garfa cada gancho (cut Leandro / cut Aumentado / os dois).
  7. Monta a Pauta da Semana no formato do `SKILL.md`.
  8. Salva a pauta como `pauta-AAAA-Wnn.md` na pasta do Drive e escreve o `ledger-AAAA-Wnn.json` da semana.
- **Estado em git (opcional):** se quiser o mesmo versionamento do `pipeline-watchdog`, a routine pode também commitar um snapshot do ledger num repo de estado privado. O Drive já é a fonte da verdade; o git é redundância/histórico.

A pauta também roda sob demanda: a qualquer momento, "rodar o radar de segunda" / "monta a pauta da semana" dispara o mesmo fluxo.

---

## 4. Como encaixa no resto (a esteira)

O Radar de Segunda é o **primeiro elo**. Ele descobre; outras skills fabricam; humano refina e publica.

A Pauta sai como documento no Drive — e é isso que costura as duas contas. O motor roda no time, mas o Leandro abre a pauta compartilhada, pega os ganchos *cut Leandro* e fabrica na conta pessoal dele (com a `conteudo-semanal`); a Amanda pega os *cut Aumentado* e fabrica no time. O documento no Drive é a ponte; nenhuma skill precisa enxergar a outra.

```
Radar de Segunda  →  Pauta da Semana (5–8 ganchos garfados)
        │
        ├─ ganchos cut Leandro   →  Leandro escolhe  →  conteudo-semanal (+ brand-leandro)  →  refina  →  publica na mão
        │                                                visual: how-brand-system
        │
        └─ ganchos cut Aumentado →  Amanda escolhe   →  trilha Aumentado (em construção)*   →  refina  →  publica na mão
                                                         visual: marca-aumentado (wordmark segurado até janeiro)
        │
        └─ (5–7 dias depois)  →  números entram no Painel  →  próxima pauta aprende
```

\* **Próximos builds combinados** (não fazem parte deste pacote, são os elos seguintes da esteira):
- **Trilha Aumentado** — skill irmã da `conteudo-semanal`, pra Amanda fabricar a peça do Aumentado a partir do gancho. Por ora, dá pra usar a `conteudo-semanal` adaptada à moldura Humano+.
- **Extensão da `conteudo-semanal`** — ensinar ela a consumir um gancho da pauta como ponto de partida (em vez de gerar pauta do zero) e a registrar o post publicado no Painel.

---

## Resumo do que você precisa fazer agora

1. ✅ ~~Criar a pasta no Drive e colar o ID~~ — **feito** (ID `1fWtiWlGgrsIomxI66EvLUUrRomx1eEEm`). Só confira que o acesso está liberado pra conta de time.
2. Criar o Painel (Sheet) com as colunas do passo 2.
3. Agendar a routine no Claude Code (ou só rodar sob demanda no começo, pra pegar o jeito).

Feito o Painel, na primeira execução ele roda em cold start (sem dado de performance) e já entrega a primeira Pauta da Semana. A partir da terceira ou quarta semana, com números no ledger, ele começa a afinar a mira.

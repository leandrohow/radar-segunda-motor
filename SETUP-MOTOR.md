# SETUP-MOTOR.md — Radar de Segunda

Motor da pauta rodando como **Claude Code remote routine**: na nuvem, agendado pra segunda de manhã (07:01), mesmo com seu laptop fechado. Roda na **conta de time** (onde a Amanda tem acesso). Versão sem X (canais duráveis + busca web).

> Irmão de produção do motor do Radar de Sábado. Mesma arquitetura (repo + Drive + routine); filtro invertido. Sábado entrega repertório pra estudar; Segunda entrega ganchos pra produzir.

---

## Arquitetura (3 estágios)

```
[ Gatilho agendado ]  segunda ~07:01, na nuvem (Anthropic-managed)
        │
        ▼
[ Subagentes em paralelo ]  1 por camada → devolvem JSON (contrato abaixo)
        │
        ▼
[ Agente editor/caçador ]  carrega a skill (cérebro) · lê o ledger + números do Drive ·
                           filtra por perna · garfa · escreve a Pauta + o ledger no Drive
```

Por que repo + Drive: a routine roda na nuvem, **clona o repositório que você configurar e usa seus conectores — não enxerga arquivos locais**. Então o cérebro (a skill) viaja dentro do repo, e a saída (a Pauta) vai pro Drive via conector.

**Saída em Markdown, não HTML — de propósito.** A Pauta é **insumo pra IA da esteira** (conteudo-semanal, trilha Aumentado): MD é o que essas skills consomem limpo. O ledger é JSON (estado de máquina). O HTML do Sábado existe porque a edição é pra ler com café; a Pauta de Segunda é pra **agir**, não pra ler — fazer HTML junto seria peso sem ganho.

---

## Pré-requisitos

- Plano pago (Team, no caso) — routines estão em research preview.
- Conector do **Google Drive** conectado na conta de time (você já tem).
- GitHub conectado na conta de time (você já tem: @leandrohow, repo `radar-segunda-motor`).
- A pasta do Drive já existe: **"Radar de Segunda"** · ID `1fWtiWlGgrsIomxI66EvLUUrRomx1eEEm`.

---

## Passo 1 — Subir o repo

Suba esta pasta inteira pro repo `radar-segunda-motor` (pode ser privado). A estrutura que importa:

```
radar-segunda-motor/
├── SETUP-MOTOR.md          (este arquivo)
└── .claude/
    └── skills/
        └── radar-de-segunda/
            ├── SKILL.md
            ├── SETUP.md
            └── references/
                ├── ganchos.md
                ├── fontes.md
                └── ledger.md
```

O caminho `.claude/skills/radar-de-segunda/` é o que o Claude Code reconhece como skill do projeto quando clona o repo. (A skill que você subiu no claude.ai do time serve pro uso sob demanda no chat; a routine usa a cópia que está **aqui no repo**.)

## Passo 2 — Drive e Painel

**Pasta do Drive — feito.** "Radar de Segunda" (ID `1fWtiWlGgrsIomxI66EvLUUrRomx1eEEm`), já cravado no `references/ledger.md`. Ela começa **vazia**: no primeiro run o motor roda em cold start e cria o primeiro `pauta-AAAA-Wnn.md` e o primeiro `ledger-AAAA-Wnn.json`. Não precisa semente. **Confira que a pasta está compartilhada com a conta de time.**

**Painel de números.** A interface manual onde você e a Amanda lançam os números dos posts (alcance, salvamentos, etc.). Pro motor ler de forma confiável na nuvem, a v1 usa o **drop em JSON** dentro da mesma pasta do Drive:

- Um arquivo `numeros-AAAA-Wnn.json` por semana, lançado 5–7 dias depois dos posts, no formato:

```json
{
  "semana": "2026-W27",
  "publicados": [
    {
      "gancho_id": "W27-1", "conta": "leandro", "plataforma": "linkedin",
      "formato": "post-curto", "url_post": "https://...", "data_post": "2026-07-01",
      "alcance": 0, "impressoes": 0, "salvamentos": 0, "comentarios": 0,
      "compartilhamentos": 0, "delta_seguidores": 0,
      "nota_perfil": "ex.: vários comentários de gestores de RH — público-alvo"
    }
  ]
}
```

(O `references/ledger.md` também descreve a Opção A — uma Google Sheet — como superfície humana mais amigável; se vocês preferirem a planilha, mantenham a planilha como rascunho e transcrevam pro JSON, ou deixem pra quando o Metricool entrar e escrever direto. Pro run autônomo, o JSON é o contrato.) Cold start: nas primeiras semanas não há `numeros-*.json` — o motor segue sem dado de performance, normal.

**Régua da meta — `baseline.json`.** Um arquivo único na mesma pasta, com os números de largada de cada conta/rede (Leandro e os handles da How/Aumentado), a meta de 6 meses (3× Leandro · 2× How) e o `atual`. É **exceção ao append-only**: mantido por humano, sobrescrito quando o `atual` muda (mensal já basta) — o motor **só lê**. Serve pra pauta dizer onde estamos vs. a meta (uma linha de progresso na leitura da semana). Suba o `baseline.json` que já foi gerado uma vez; depois é só atualizar o `atual` de tempos em tempos.

A pasta, no regime normal, guarda: `pauta-AAAA-Wnn.md` (motor cria) · `ledger-AAAA-Wnn.json` (motor cria) · `numeros-AAAA-Wnn.json` (vocês preenchem) · `baseline.json` (único, humano mantém).

## Passo 3 — Criar a routine

1. Vá em **claude.ai/code/routines** (ou `/schedule` no CLI; no Desktop, **Schedule → New task → New remote task**).
2. **Repositório:** aponte pro `radar-segunda-motor`.
3. **Conectores:** habilite o **Google Drive**; mantenha a busca web. Tire o resto (menos ferramenta, menos risco).
4. **Gatilho:** agendamento **semanal — segunda, 07:01** (seu fuso, SP). Em cron: `1 7 * * 1`. (O 07:01 evita o jitter pior do agendador; pode haver um atraso determinístico de alguns minutos — pra uma pauta de segunda de manhã, irrelevante.)
5. **Prompt:** cole o bloco da próxima seção.
6. Salve. A routine aparece na lista e roda sozinha na próxima segunda.

> A routine pertence à **sua** conta (não é compartilhada com a Amanda) e age como você. Tudo bem: ela escreve a Pauta na pasta **compartilhada** do Drive, e a Amanda lê de lá. A pasta é a ponte; a Amanda só não edita a routine em si.

---

## O prompt da routine

> Cole isto como o prompt da routine. Os `{{...}}` o Claude resolve no run.

```text
Você vai rodar a Pauta da Semana do Radar de Segunda. Hoje é {{data de hoje}}, semana ISO {{ano-Wnn}}.

Use a skill do projeto em .claude/skills/radar-de-segunda/ como o cérebro. Leia o SKILL.md, o references/ganchos.md, o references/aumentado.md, o references/fontes.md e o references/ledger.md ANTES de tudo, e siga as regras deles (a inversão vs. Sábado, as alavancas de perna, a lente do Aumentado, o filtro anti-slop, o garfo, o formato da Pauta, a disciplina de ledger). Lembre da regra de ouro: o ingrediente que faz crescer é o ÂNGULO, não o link.

PASSO 1 — Histórico, números e régua da meta.
Na pasta "Radar de Segunda" do Google Drive (ID 1fWtiWlGgrsIomxI66EvLUUrRomx1eEEm), leia TODOS os ledger-*.json (o estado é a união deles). Leia também os numeros-*.json que ainda não foram incorporados e grave esses números nos posts correspondentes do ledger (publicados). Leia também o baseline.json (régua fixa da meta de 6 meses) — guarde base/atual/meta por conta e rede pra escrever a linha de progresso na pauta. Se a pasta estiver vazia (ou sem numeros-*.json / sem baseline.json), trate como cold start e omita o que não existir. Do ledger, extraia: o que performou (tema/formato/alavanca), o que cansou, e ganchos escolhidos que ainda não viraram post.

PASSO 2 — Varredura (subagentes em paralelo).
Dispare um subagente por camada (ver fontes.md). Cada um varre e devolve SÓ um JSON no "Contrato do subagente" abaixo — sem prosa. NÃO há X nesta versão: ignore o X e use os canais duráveis dos curadores + busca web.
- Substância: labs, GitHub Trending, papers — o lastro (fato real por trás de um possível gancho).
- Conversa-Curadores: o que os curadores estão DEBATENDO e amplificando; convergência de vários no mesmo tema = onda quente.
- Conversa-Nicho: o discurso de IA aplicada / futuro do trabalho que está pegando (o take que repercutiu, a pergunta que gerou comentário, o formato que estourou).

PASSO 3 — Edição (você como editor/caçador).
Junte os achados, deduplique por URL. Filtre cada candidato por PERNA (nomeie a alavanca do ganchos.md — se não dá pra nomear, não é gancho, é notícia: descarte) e por NOVIDADE (contra o ledger: não repita ângulo já registrado), e pelo filtro anti-slop e pelo teste "só o Leandro / só o Aumentado diria isso assim?". Cruze com o que o ledger disse que performou. Selecione de 5 a 8 ganchos.

PASSO 4 — Garfo, prioridade e montagem.
Garfe cada gancho: cut Leandro / cut Aumentado / os dois (se "os dois", escreva a linha que diferencia as pegadas — pra Amanda e Leandro não colidirem). Para o cut Aumentado, angule pela tese real do references/aumentado.md (use as frases-cânone ★ como sementes; atribua os conceitos de fronteira corretamente). Marque como Candidato ao Dossiê o sinal que confirma/estende/contradiz a tese ou é achado de fronteira novo. Dê a cada gancho uma PRIORIDADE (0–100) conforme references/ganchos.md (força/número de alavancas + convergência + aderência ao que o ledger diz que performou + timing perecível) e ordene os ganchos da maior prioridade pra menor. Monte a Pauta no formato EXATO do SKILL.md: leitura da semana (2-3 linhas — e, se o baseline.json mostrar movimento desde a base, inclua UMA linha de progresso rumo à meta, ex.: "Aumentado/LinkedIn: 32,8k → 34,1k · 52% da meta"), a linha "o que funcionou" (do ledger, se houver dado), os ganchos ordenados por prioridade (sinal+link, por que tem perna, ângulo, garfo, formato sugerido, prioridade), os Descartados da semana, os Candidatos a Fonte e os Candidatos ao Dossiê.

PASSO 5 — Escrita no Drive (pasta "Radar de Segunda").
- Crie o arquivo pauta-{{ano-Wnn}}.md com a Pauta da Semana.
- Crie o arquivo ledger-{{ano-Wnn}}.json com todos os ganchos da semana (escolhidos e não escolhidos, no schema do ledger.md) e os publicados conhecidos. (Append-only: NÃO sobrescreva os ledgers antigos; crie o desta semana.)

PASSO 6 — Resumo.
Devolva 2-3 linhas: quantos ganchos entraram e o garfo (quantos Leandro / Aumentado / os dois), se a semana ficou magra, e se entrou número novo no ledger nesta rodada.
```

---

## Contrato do subagente

Cada subagente de camada devolve **exatamente** este JSON (e nada mais). É o que o editor consome.

```json
{
  "camada": "Substancia | Conversa-Curadores | Conversa-Nicho",
  "items": [
    {
      "title": "título curto do achado",
      "url": "https://fonte-real",
      "source": "quem publicou (lab, autor, veículo, perfil)",
      "date": "AAAA-MM-DD",
      "alavanca": "tensao | status | medo-alivio | utilidade | numero | trincheira | reframe | convergencia",
      "signal": "convergencia | primaria | trending",
      "nota": "uma linha: por que tem perna OU o ângulo bruto sugerido"
    }
  ]
}
```

Regras do contrato:
- `url` sempre real e verificável. Sem URL, o item não existe.
- `date` é a data real do fato (não da repostagem). Agregador reembala notícia velha — confira.
- `alavanca` tem que ser nomeável. Se o subagente não consegue dizer qual alavanca o item puxa, ele NÃO é gancho — é matéria de estudo (manda pro Sábado, não pra cá).
- `signal: "convergencia"` quando vários curadores/fontes tocam o mesmo tema na mesma semana — é o sinal de onda quente, sobe na pauta.

---

## Escopo da v1 (sem X)

Os curadores que vivem só no X entram de forma parcial nesta versão — só o que vaza pros canais duráveis ou pra busca web. Como boa parte do "pulso" do nicho vive em LinkedIn/IG e em X, a camada Conversa-Nicho é a mais limitada por código nesta fase. É proposital: fazer o pipeline rodar ponta a ponta primeiro. O X/redes oficial entra numa fase seguinte (e o ledger de performance já começa a compensar — o que funciona nas SUAS contas vira o melhor sinal, independente de fonte externa).

## Ressalvas honestas

- Routines são **research preview** — comportamento e disponibilidade podem mudar. Valide os primeiros runs.
- **Rede:** o ambiente Default da nuvem usa allowlist de domínios confiáveis; hosts fora dela tomam 403. A camada Substância (labs/GitHub/papers) costuma cair em domínios conhecidos; a Conversa-Nicho varre redes/discurso e é onde isso mais pode esbarrar. Como o motor do **Sábado já varre e grava no Drive** nessa mesma infra, use a mesma config de ambiente/rede dele — e confirme no primeiro run o que a varredura alcança.
- **Disciplina do Painel:** se os `numeros-*.json` não entram, o motor fica cego e vira gerador de pauta genérica. É a única tarefa semanal inegociável — sua e da Amanda. Cold start nas primeiras 3-4 semanas é esperado.
- **O primeiro run é o teste real.** Abra a sessão da routine na barra lateral pra ver o que o Claude fez e ajustar o prompt se precisar.

## Próxima fase (opcional)

Mesmo repo, evolução natural: plugar o **Metricool** (ou similar) escrevendo os números direto — aposenta o drop manual de `numeros-*.json`. E, se quiser, um orquestrador em Python no GitHub Actions reusando o contrato do subagente acima, com a skill seguindo como cérebro. Por ora: manual, e o ledger aprende sozinho a cada semana.

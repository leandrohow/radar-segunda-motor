# Ledger — memória e performance

O ledger é o que separa **fábrica de conteúdo** de **máquina de crescimento**. Sem ele, toda segunda você caça do zero e repete ângulo. Com ele, a pauta de cada semana aprende com o que a anterior entregou.

Ele faz duas coisas:
1. **Não repetir** — registra todo gancho proposto e todo post publicado, pra você não voltar no mesmo ângulo.
2. **Aprender** — registra os números de cada post, pra você enviesar a próxima pauta rumo ao que funcionou.

Leia o ledger **antes** de montar a pauta. Escreva nele **depois**. Os dois passos são inegociáveis — é o que o Radar de Sábado faz, e por isso ele não se repete.

---

## Configuração do Drive (fixa — não buscar)

> ✅ **Configurado.** Pasta criada e ID cravado abaixo. (URL: https://drive.google.com/drive/folders/1fWtiWlGgrsIomxI66EvLUUrRomx1eEEm) — esta pasta deve viver no Drive da conta de time, onde a Amanda tem acesso.

Pasta do Radar de Segunda no Drive: **"Radar de Segunda"**, ID: `1fWtiWlGgrsIomxI66EvLUUrRomx1eEEm`

**Não use busca por nome ou fullText pra achar a pasta ou o ledger** — a busca do Drive não indexa JSON e retorna vazio (já falhou em produção no Radar de Sábado). Em vez disso:
- Listar a pasta: `api_query` = `'1fWtiWlGgrsIomxI66EvLUUrRomx1eEEm' in parents`, com `order_by: createdTime desc`.
- Ledger: arquivos versionados append-only no padrão `ledger-AAAA-Wnn.json` (o conector só tem `create_file`, sem update). O estado atual = união de todos os ledgers, do mais antigo ao mais novo. Leia todos antes de montar; ao fim, crie o `ledger-AAAA-Wnn.json` da semana corrente.
- Se a listagem voltar vazia (primeira execução), trate como ledger limpo e siga em cold start.
- **Baseline (régua da meta):** um arquivo único `baseline.json` na mesma pasta (exceção ao append-only — é mantido por humano, sobrescrito quando o `atual` muda; o motor **só lê**). Traz `base`, `meta_abs`/`meta_mult` e `atual` por conta/rede. Leia-o e, na leitura da semana da pauta, escreva **uma linha de progresso** rumo à meta quando houver movimento (ex.: "Aumentado/LinkedIn: 32,8k → 34,1k · 52% da meta de 65,6k"). Se o arquivo não existir, omita a linha de progresso — não invente número.

---

## O Painel de números (a interface manual)

Por ora os números entram **na mão** — você e a Amanda lançam, a skill lê. Quando plugar a ferramenta (Metricool ou similar), ela passa a alimentar o mesmo lugar e o resto não muda.

**O contrato:** existe um lugar único onde os números dos posts publicados são lançados. A skill lê esse lugar no passo 2 do fluxo, casa cada linha com o post no ledger e grava os números.

Duas formas de operar o Painel (escolha uma no `SETUP.md`):

- **Opção A — Planilha (recomendada):** uma Google Sheet "Painel — Radar de Segunda" com uma linha por post. Colunas: `conta` (leandro/aumentado), `plataforma` (instagram/linkedin), `url_post`, `data_post`, `gancho_id` (opcional), `alcance`, `impressoes`, `salvamentos`, `comentarios`, `compartilhamentos`, `delta_seguidores`, `nota_perfil`. Friendly pra preencher; é o que a ferramenta substitui depois.
- **Opção B — Drop no Drive:** um arquivo `numeros-AAAA-Wnn.json` por semana na pasta, no mesmo padrão append-only do ledger. Mais consistente com o resto, menos amigável de digitar.

**Sempre lance os números 5–7 dias após o post** (quando alcance e salvamento já estabilizaram). Post de ontem ainda não tem leitura.

Se não houver números novos numa semana (esquecimento, semana corrida): siga em cold start parcial e registre na leitura da semana que a pauta foi sem dado fresco. **Não invente número.**

---

## Schema do ledger

```json
{
  "semana": "2026-W27",
  "data_pauta": "2026-06-29",
  "ganchos": [
    {
      "id": "W27-1",
      "titulo": "Prompt engineering virou invisível",
      "sinal_url": "https://...",
      "tema": "habilidades",
      "alavanca": ["contrarian", "status"],
      "garfo": "os dois",
      "prioridade": 86,
      "escolhido": true,
      "conta": ["leandro", "aumentado"],
      "status": "ideia"
    }
  ],
  "publicados": [
    {
      "gancho_id": "W27-1",
      "conta": "leandro",
      "plataforma": "linkedin",
      "formato": "post-curto",
      "data_post": "2026-07-01",
      "url_post": "https://...",
      "numeros": {
        "alcance": 0,
        "impressoes": 0,
        "salvamentos": 0,
        "comentarios": 0,
        "compartilhamentos": 0,
        "delta_seguidores": 0,
        "nota_perfil": "ex.: vários comentários de gestores de RH — público-alvo"
      },
      "lido_em": "2026-07-08"
    }
  ]
}
```

- `ganchos` registra a pauta inteira (escolhidos e não escolhidos) — pra não repetir ângulo e pra você ver depois o que virou post. `prioridade` (0–100, ver `ganchos.md`) entra junto.
- `status` é **opcional** — um campo leve de produção (`ideia` → `rascunho` → `revisão` → `publicado`) pra quem quiser rastrear o ciclo da peça. Não é obrigatório; este sistema foi desenhado pra ser leve, não um gestor de tarefas. Ative só se a Amanda sentir falta.
- `publicados` começa só com os campos de identificação; os `numeros` entram quando o Painel é lido (podem entrar num ledger de semana posterior, já que a leitura é 5–7 dias depois).

---

## Métricas que importam (e a recomposição de audiência)

Seguir contando seguidor não basta — principalmente na conta do Aumentado.

- **Para o Leandro:** a meta é **triplicar a base em 6 meses**. Salvamento e compartilhamento são os melhores preditores de alcance novo; comentário sinaliza take que pegou; `delta_seguidores` é o placar. Priorize ganchos cujas alavancas históricas trouxeram seguidor, não só curtida.
- **Para o Aumentado (nos handles da How):** a base atual é **legado B2C que não converte**, e o handle vira Aumentado em janeiro. Aqui o objetivo real não é número bruto — é **recompor** a audiência rumo ao profissional-alvo do Aumentado. Por isso `nota_perfil` importa tanto quanto `delta_seguidores`: registre QUEM está reagindo e seguindo (cargo, setor, perfil aparente nos comentários). **Alguma evasão do legado é saudável** — não é perda, é troca de público. Uma semana que perde 20 seguidores curiosos e ganha 15 do perfil-alvo é uma boa semana, e o ledger precisa enxergar isso.

Meta de referência: Leandro Insta 3.215 / LinkedIn 10,7k; How Insta 24,1k / LinkedIn 32,8k (ponto de partida — atualize quando confirmar). Dobrar 32,8k no orgânico é a montanha mais íngreme; instrumente expectativa por isso.

---

## Como o ledger enviesa a caça

Ao ler o ledger no início, extraia três coisas e use na pauta:
1. **O que performou** — tema/formato/alavanca com melhor salvamento/compartilhamento/seguidor. Traga mais do mesmo veio (sem repetir o ângulo exato).
2. **O que cansou** — ângulo ou tema que já apareceu demais, ou que performou mal. Evite.
3. **O que ficou pendente** — gancho escolhido que ainda não virou post. Lembre na leitura da semana, sem reabrir como novo.

Na pauta, a linha "O que funcionou" deve refletir explicitamente o item 1 — é o que justifica as escolhas da semana para o Leandro e a Amanda.

---

## Atualização do histórico

**Passo final, inegociável.** Depois de montar a pauta, crie o `ledger-AAAA-Wnn.json` da semana com todos os ganchos (escolhidos e não) e os `publicados` conhecidos. Sem isso, a pauta seguinte fica cega e repete tudo. Confirme que escreveu antes de encerrar.

# OneClick Ads Skills — Meta Ads, Google Ads e MCP para Claude

Skills que ensinam o Claude (e outros assistentes compatíveis) a planejar e criar
campanhas de **Meta Ads** e **Google Ads** com estruturas validadas e a calcular as
métricas de tráfego pago. As três primeiras servem para qualquer pessoa, com ou sem a
OneClick Ads; as outras operam a [OneClick Ads](https://oneclickads.com.br) pelo
**MCP da OneClick Ads**.

_English below._

## Instalar

**No Claude Code**, como plugin. Escolha um:

```
/plugin install ads-playbook --marketplace oneclickads/skills
```

```
/plugin install oneclick-ads --marketplace oneclickads/skills
```

- `ads-playbook`: as três skills abertas, para qualquer pessoa.
- `oneclick-ads`: o MCP da OneClick Ads já configurado (pede login na OneClick no
  primeiro uso). O próprio MCP entrega as sete skills ao Claude, sempre na versão mais
  recente, então o plugin não as copia. Já inclui as skills do `ads-playbook`, então
  instale só um.

**No Claude (claude.ai e app)** ou em qualquer assistente compatível com skills via MCP:
conecte o MCP da OneClick Ads e as sete skills chegam junto, sem instalar nada.

Em versões do Claude Code anteriores à 2.1.275, adicione o marketplace antes:
`/plugin marketplace add oneclickads/skills` e depois
`/plugin install ads-playbook@oneclickads` (ou `oneclick-ads@oneclickads`).

**Em outros assistentes** (Cursor, Codex e outros que leem skills de arquivos):

```bash
npx skills add oneclickads/skills
```

## Para qualquer pessoa

Funcionam sem nenhuma ferramenta extra: o assistente monta o plano da campanha para você
seguir no Gerenciador de Anúncios ou no Google Ads. Se ele tiver acesso à sua conta (pelo
MCP da OneClick Ads ou outro), usa esse acesso.

| Skill                                                                           | O que o assistente faz                                                                                                                       |
| ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| [`meta-ads-campaign-structures`](skills/meta-ads-campaign-structures/SKILL.md)   | Escolhe a estrutura validada (Andrômeda, CBO, ABO, teste de criativos, remarketing), calcula o orçamento e entrega o plano da campanha Meta |
| [`google-ads-campaign-planning`](skills/google-ads-campaign-planning/SKILL.md)   | Escolhe entre Pesquisa, Performance Max, Demand Gen, Display, Shopping e App e estrutura palavras-chave, negativas, anúncios e lances        |
| [`ads-metrics`](skills/ads-metrics/SKILL.md)                                     | Explica e calcula ROAS, POAS, CPA, CAC, LTV, CTR, CPM e define o CPA máximo e o ROAS de equilíbrio do negócio                               |

Exemplos:

- "Tenho 12 criativos e vendo um produto de R$ 197. Qual estrutura uso no Meta Ads e quanto invisto por dia?"
- "Monte uma campanha de Pesquisa no Google Ads para uma clínica odontológica em Campinas."
- "Minha margem é 35%. Qual o ROAS de equilíbrio e o CPA máximo?"

## Com a OneClick Ads

Pré-requisito: o MCP da OneClick Ads (`https://mcp.oneclickads.com.br/mcp`) conectado com
escopo de leitura e escrita. Passo a passo:
[Conectar um assistente de IA](https://docs.oneclickads.com.br/pt-br/mcp/connect-an-ai-assistant/).

| Skill                                                                       | O que o assistente faz                                                                                               |
| --------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [`oneclick-ads-onboarding`](skills/oneclick-ads-onboarding/SKILL.md)         | Conecta o Meta, ativa a conta de anúncios, liga o OneClick Tracking, cria a primeira otimização e faz um diagnóstico |
| [`meta-ads-campaign-builder`](skills/meta-ads-campaign-builder/SKILL.md)     | Cria a campanha Meta na sua conta a partir de uma estrutura validada e dos seus melhores criativos                   |
| [`google-ads-campaign-builder`](skills/google-ads-campaign-builder/SKILL.md) | Cria a campanha de Google Ads na sua conta, de qualquer um dos seis tipos                                            |
| [`optimization-setup`](skills/optimization-setup/SKILL.md)                   | Escolhe o perfil, calcula o CPA para pausar ou o ROAS mínimo pelo histórico e ativa a otimização automática           |

Exemplos:

- "Monte uma Escala Andrômeda (1-1-10) com meus 10 melhores anúncios dos últimos 90 dias, R$ 150/dia."
- "Configure a otimização dessa campanha nova com o padrão da conta."
- "Iniciar onboarding na OneClick Ads."

Toda campanha sobe **pausada**, e o assistente mostra o resumo e pede confirmação antes
de criar qualquer coisa.

## Fonte

As estruturas vêm das [Campanhas Validadas](https://docs.oneclickads.com.br/pt-br/validated-campaigns/)
e as regras de criação e otimização da [documentação da OneClick Ads](https://docs.oneclickads.com.br/pt-br/).

---

## English

Skills that teach Claude (and other compatible assistants) to plan and build **Meta Ads**
and **Google Ads** campaigns with proven structures and to work out paid-traffic metrics.

- **For everyone, no extra tools needed**: `meta-ads-campaign-structures`,
  `google-ads-campaign-planning`, `ads-metrics`. The assistant delivers a build sheet for
  Ads Manager or Google Ads, or uses whatever ad-account tools it has.
- **With OneClick Ads** (MCP at `https://mcp.oneclickads.com.br/mcp`, read and write
  scope): `oneclick-ads-onboarding`, `meta-ads-campaign-builder`,
  `google-ads-campaign-builder`, `optimization-setup`. Campaigns are created **paused**
  after the assistant shows a summary and gets your confirmation.

Install in Claude Code with `/plugin install ads-playbook --marketplace oneclickads/skills`
(open skills) or `/plugin install oneclick-ads --marketplace oneclickads/skills` (the
OneClick Ads MCP, which serves all seven skills itself, always current; install one, not
both). In Claude (claude.ai and the app) and any assistant that supports skills over MCP,
connecting the OneClick Ads MCP is enough. Assistants that read skills from files:
`npx skills add oneclickads/skills`. Sources: the
[OneClick Ads Validated Campaigns](https://docs.oneclickads.com.br/en/validated-campaigns/)
and the [OneClick Ads docs](https://docs.oneclickads.com.br/en/).

## Licença / License

[MIT](LICENSE)

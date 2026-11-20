# OneClick Ads Skills — MCP Meta Ads e Google Ads para Claude

Skills que ensinam o Claude (e outros assistentes compatíveis) a operar a
[OneClick Ads](https://oneclickads.com.br) pelo **MCP da OneClick Ads**: criar campanhas
de **Meta Ads** com as estruturas validadas (Escala Andrômeda, CBO, ABO, teste de
criativos), criar campanhas de **Google Ads** (Pesquisa, Performance Max, Demand Gen,
Display, Shopping e App), configurar as otimizações e fazer o onboarding da conta.

_English below._

## Instalar

```bash
npx skills add oneclickads/skills
```

Pré-requisito: o MCP da OneClick Ads conectado ao seu assistente, no endereço
`https://mcp.oneclickads.com.br/mcp`, com escopo de **leitura e escrita** para criar
campanhas. Passo a passo:
[Conectar um assistente de IA](https://docs.oneclickads.com.br/pt-br/mcp/connect-an-ai-assistant/).

## Skills

| Skill                                                                     | O que o assistente faz                                                                                              |
| ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| [`oneclick-ads-onboarding`](skills/oneclick-ads-onboarding/SKILL.md)       | Conecta o Meta, ativa a conta de anúncios, liga o OneClick Tracking, cria a primeira otimização e faz um diagnóstico |
| [`meta-ads-campaign-builder`](skills/meta-ads-campaign-builder/SKILL.md)   | Cria a campanha Meta a partir de uma estrutura validada e dos melhores criativos da conta, pausada para revisão       |
| [`google-ads-campaign-builder`](skills/google-ads-campaign-builder/SKILL.md) | Cria Pesquisa, Performance Max, Demand Gen, Display, Shopping ou App, pausada para revisão                         |
| [`optimization-setup`](skills/optimization-setup/SKILL.md)                 | Escolhe o perfil, calcula CPA para pausar ou ROAS mínimo e orçamento máximo pelo histórico, e ativa as otimizações  |

Exemplos do que pedir:

- "Monte uma Escala Andrômeda (1-1-10) com meus 10 melhores anúncios dos últimos 90 dias, R$ 150/dia, pausada."
- "Quero testar estes 4 criativos: [links]. Qual estrutura você recomenda?"
- "Crie uma campanha de Pesquisa no Google Ads para [produto] em São Paulo, maximizando cliques."
- "Configure a otimização dessa campanha nova com o padrão da conta."
- "Iniciar onboarding na OneClick Ads."

Toda campanha sobe **pausada**, e o assistente mostra o resumo e pede confirmação antes
de criar qualquer coisa.

## Fonte

As skills seguem a [documentação da OneClick Ads](https://docs.oneclickads.com.br/pt-br/)
e apontam para as páginas dela; o assistente lê a página com a ferramenta `getDocsPage`
do próprio MCP quando precisa de detalhe.

---

## English

Skills that teach Claude (and other compatible assistants) to run
[OneClick Ads](https://oneclickads.com.br) through the **OneClick Ads MCP**: build
**Meta Ads** campaigns from validated structures (Andromeda Scale, CBO, ABO, creative
testing), build **Google Ads** campaigns (Search, Performance Max, Demand Gen, Display,
Shopping, App), set up OneClick optimizations and onboard an account.

Install with `npx skills add oneclickads/skills`. Requires the OneClick Ads MCP
(`https://mcp.oneclickads.com.br/mcp`) connected with read and write scope — see
[Connect an AI assistant](https://docs.oneclickads.com.br/en/mcp/connect-an-ai-assistant/).

Every campaign is created **paused**, and the assistant shows a summary and asks for
confirmation before creating anything. The skills follow the
[OneClick Ads docs](https://docs.oneclickads.com.br/en/) and point to their pages, which the
assistant reads with the MCP's `getDocsPage` tool.

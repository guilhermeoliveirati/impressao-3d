# MEMÓRIA DO PROJETO — Painel Impressão 3D

> Arquivo para orientar qualquer IA ou pessoa que for mexer neste projeto. Leia inteiro antes de alterar algo.
> Pode virar repositório **público**: não coloque aqui dados de clientes, chaves, senhas nem e-mails.

## Origem

Cópia adaptada do **Painel da Oficina** (`Meu Drive/Mecanica/Painel`, v1.9.7): mesma arquitetura, mesmo visual,
mesmas abas de Despesas, Clientes, Dízimo, Resumo e Config. A diferença é o negócio: aqui se vendem **peças impressas em 3D**,
e existe a aba **Produtos**, que calcula o custo de produção e o preço de venda.

## Onde fica

Igual à Oficina: código e dados na mesma pasta do Google Drive; o git só leva o código.

- Pasta: `Meu Drive/Impressora 3D` do Google Drive da conta do projeto (no computador, `H:\Meu Drive\Impressora 3D`). É o repositório git.
- Código: GitHub `guilhermeoliveirati/painel-impressao-3d` (público, branch `main`).
- Site: `https://guilhermeoliveirati.github.io/painel-impressao-3d/painel.html` (GitHub Pages, `git push` publica em ~1 minuto).
- Dados: `dados.json` nessa pasta do Drive, compartilhado por todos os aparelhos (computador abre pela pasta, tablet/celular pelo Google Drive). Nunca no git.

## Regras obrigatórias (iguais às da Oficina)

1. **Toda alteração aumenta a versão** (`VERSAO` em `painel.html`) e ganha uma linha em **Versões**.
   Correção `x.x.+1`; recurso novo `x.+1.0`; quebra do formato do `dados.json` `+1.0.0`.
2. **Nunca enviar dados para o GitHub**: `dados.json`, `.xlsx`, `.csv` estão no `.gitignore`.
3. **Nunca gravar chaves no código.** A chave da Gemini fica no `dados.json` (`gemini.chave`).
   As únicas credenciais no código são as públicas do Google (`GOOGLE` em `painel.html`).
4. Interface em **português do Brasil**, letras em `rem`, **sem CSS `zoom`**, valores com `brl()`.
5. A venda **nunca é salva sozinha**: voz/Gemini só preenchem o formulário.
6. Antes de mexer no `dados.json` por fora do painel, fazer cópia de segurança.
7. `localStorage` usa o prefixo `p3d.` e o IndexedDB se chama `painel-3d`, para não misturar com a Oficina se os dois
   ficarem no mesmo endereço do GitHub Pages.

## Formato do `dados.json`

```jsonc
{
  "versao": 1, "atualizado": "…", "proximoId": 10,     // ids: v vendas, d despesas, p produtos, i impressoras, f filamentos
  "taxas": { "Pix": 0, "Débito": 0.0137, "Crédito 1x": 0.0315, … },
  "config": { "energiaKwh": 0.95, "maoDeObraHora": 25, "falhaPct": 10, "margemPct": 100, "arredondar": "90" },  // arredondar: nao | 1 | 90 | 5
  "impressoras": [ { "id", "nome", "potenciaW", "valor", "vidaUtilH", "manutHora" } ],
  "filamentos":  [ { "id", "nome", "material", "precoKg" } ],
  "produtos": [ { "id", "nome", "categoria", "impressoraId", "horas", "minutos", "unidades", "maoMin", "extras",
                  "falhaPct": null, "margemPct": null, "precoVenda": null,   // null = usa o padrão de Config / o preço sugerido
                  "filamentos": [ { "filamentoId", "g" } ], "obs" } ],
  "vendas": [ { "id", "data", "cliente", "itens": [ { "produtoId"|null, "nome", "qtd", "preco", "custo" } ],
                "frete", "cobrado", "metodo", "custo", "taxa", "lucro", "obs" } ],
  "despesas": [ { "id", "data", "categoria", "descricao", "valor" } ],
  "dizimos": { "AAAA-MM": { "itens": [ { "nome", "valor" } | { "nome", "auto": true } ] } },
  "gemini": { "chave": "…", "modelo": "…" }   // opcional
}
```

## Regras de negócio

**Custo de uma impressão** (`custoProduto`):
- filamento = Σ gramas × preço do kg ÷ 1000
- energia = potência (W) ÷ 1000 × horas × R$/kWh
- desgaste e manutenção = horas × (valor da impressora ÷ vida útil em horas + manutenção por hora)
- perdas = (filamento + energia + desgaste) × % de perda (produto ou padrão)
- mão de obra = minutos de trabalho manual ÷ 60 × R$/hora
- **custo por peça** = custo da impressão ÷ peças por impressão + extras por peça

**Preço sugerido** = custo por peça × (1 + margem%), arredondado conforme Config.
**Preço de venda** = `precoVenda` do produto, ou o sugerido se estiver vazio.
A tabela "Preço por forma de pagamento" mostra o preço para receber o mesmo que no Pix: preço ÷ (1 − taxa).

**Venda**: vários itens (do catálogo ou avulsos). Ao escolher um produto, preço e custo vêm do catálogo e ficam
**gravados na venda** (mudar custo/preço depois não altera vendas antigas).
- Valor cobrado = soma dos itens + frete, editável (desconto ou valor a mais).
- Taxa = cobrado × taxa do método.
- **Lucro da venda = cobrado − custo de produção dos itens − taxa.**
- Mão de obra entra como custo; quem quiser que conte como lucro coloca R$ 0/h em Config.

**Despesas**: compra de filamento **não** é despesa (já entra no custo de cada venda). Fixas copiáveis: MEI, Aluguel, Internet.
**Dízimo e Resumo**: mesmas regras da Oficina; o item automático se chama "Impressão 3D" (= lucro − despesas do mês).

## Abas

Geral (KPIs, gráfico mensal, produtos mais vendidos, formas de pagamento, mapa mês × ano, fechamento) ·
Vendas (itens, pagamento em botões, voz/Gemini, lista do mês, Excel) · **Produtos** (calculadora + catálogo) ·
Despesas · Clientes · Dízimo · Resumo · Config (custos de produção, impressoras, filamentos, taxas, Gemini, letra, exportar).

## Como testar sem mexer nos dados reais

Copiar `painel.html` + `dados.json` para uma pasta temporária, `python -m http.server 8765` e no console:
`D = await fetch('dados.json').then(r=>r.json()); iniciar();` — sem "fonte", nada é gravado.
Conferir sintaxe extraindo o `<script>` e rodando `node --check`.

## Google Cloud (criado em 2026-11-05)

- Projeto `painel-impressao-3d` (número 746748382684), na conta dona do Drive do painel. Drive API e Picker API ativas.
- Tela de consentimento "Externo", em modo **Testando**: só os e-mails da lista "Usuários de teste" conseguem entrar
  (hoje, só a conta do projeto). Para outra pessoa usar, adicionar o e-mail dela em Google Auth Platform › Público-alvo.
- Cliente OAuth "Painel Impressao 3D Web": origens `http://localhost:8765` e `https://guilhermeoliveirati.github.io`.
  Se o GitHub Pages ficar em outra conta do GitHub, adicionar a nova origem no cliente e na chave.
- Chave "Picker - Painel Impressao 3D": restrita à Picker API e aos sites acima (o console incluiu também a Service Management API, que não dá acesso a dados).

## Pendências

- Colocar o `dados.json` no Google Drive da conta do projeto e testar o login no tablet/celular.


## Versões

| Versão | Data | Mudança |
|---|---|---|
| 1.0.0 | 2026-11-05 | Primeiro modelo, a partir do Painel da Oficina 1.9.7: aba Produtos (custo de filamento, energia, desgaste, perdas, mão de obra, extras; preço sugerido e por forma de pagamento), vendas com vários itens e custo gravado, impressoras e filamentos em Config, Geral com produtos mais vendidos |
| 1.0.1 | 2026-11-05 | Código e dados passam para a pasta Meu Drive › Impressora 3D (Drive da conta do projeto); tela de abertura e seletor do Drive indicam essa pasta |
| 1.0.2 | 2026-11-05 | Atalho no celular: ícones PNG 192/512 (e maskable) no manifest, que o Chrome do Android exige para instalar; eixo do gráfico sem vendas não repete mais "R$ 1" |

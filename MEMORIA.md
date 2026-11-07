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
- Código: GitHub `guilhermeoliveirati/impressao-3d` (público, branch `main`).
- Site: `https://guilhermeoliveirati.github.io/impressao-3d/painel.html` (GitHub Pages, `git push` publica em ~1 minuto).
- O endereço antigo (`/painel-impressao-3d/`) foi abandonado na 1.0.4: o Chrome do Android guardou uma instalação quebrada presa a ele.
  Se a instalação no celular travar de novo ("Não foi possível abrir o aplicativo"), a saída é a mesma: renomear o repositório.
- Dados: `dados.json` nessa pasta do Drive, compartilhado por todos os aparelhos (computador abre pela pasta, tablet/celular pelo Google Drive). Nunca no git.

## Regras obrigatórias (iguais às da Oficina)

1. **Toda alteração aumenta a versão** (`VERSAO` em `painel.html`) e ganha uma linha em **Versões**.
   Correção `x.x.+1`; recurso novo `x.+1.0`; quebra do formato do `dados.json` `+1.0.0`.
2. **Nunca enviar dados para o GitHub**: `dados.json`, `.xlsx`, `.csv` estão no `.gitignore`.
3. **Nunca gravar chaves no código.** As únicas credenciais no código são as públicas do Google (`GOOGLE` em `painel.html`).
4. Interface em **português do Brasil**, letras em `rem`, **sem CSS `zoom`**, valores com `brl()`.
5. A venda **nunca é salva sozinha**: o orçamento aprovado só preenche o formulário; quem salva é o usuário.
   Não usar `confirm()`/`alert()` do navegador (no app instalado podem não aparecer e os botões "não funcionam"):
   usar `confirmar()` e `avisar()` do próprio painel.
6. Antes de mexer no `dados.json` por fora do painel, fazer cópia de segurança.
7. **Toda versão ganha uma entrada no topo de `NOVIDADES`** (em `painel.html`), em linguagem simples, com `novo`
   (o que mudou) e `uso` (como usar). O painel mostra essa janela uma vez em cada aparelho depois da atualização.
   Se a mudança criar ou mudar um passo do uso básico, atualizar também `PASSOS` (o passo a passo do primeiro acesso).
8. `localStorage` usa o prefixo `p3d.` e o IndexedDB se chama `painel-3d`, para não misturar com a Oficina se os dois
   ficarem no mesmo endereço do GitHub Pages.

## Formato do `dados.json`

```jsonc
{
  "versao": 1, "atualizado": "…", "proximoId": 10,     // ids: v vendas, d despesas, p produtos, i impressoras, f filamentos
  "taxas": { "Pix": 0, "Débito": 0.0137, "Crédito 1x": 0.0315, … },
  "config": { "energiaKwh": 0.95, "maoDeObraHora": 25, "falhaPct": 10, "margemPct": 100, "arredondar": "90" },  // arredondar: nao | 1 | 90 | 5
  "impressoras": [ { "id", "nome", "potenciaW", "valor", "vidaUtilH", "manutHora" } ],
  "filamentos":  [ { "id", "nome", "material", "precoKg" } ],
  "proximoCliente": 3,                // próximo código de cliente (#001, #002…)
  "clientes": [ { "id", "codigo", "nome", "telefone" /* só números */ } ],
  "parcelamentos": [ { "id", "descricao", "total", "parcelas", "inicio": "AAAA-MM", "pessoas": ["Eu", …], "pagos": { "AAAA-MM": [true, false, …] } } ],
  "orcamentos": [ { "id", "data", "cliente", "validade" /* dias */, "itens": [ …iguais aos da venda… ], "frete", "valor", "custo", "obs",
                    "status": "aberto" | "aprovado" | "recusado", "vendaId": null, "clienteId" } ],   // item pode ter "detalhe" (peça sob medida)
  "produtos": [ { "id", "nome", "categoria", "impressoraId", "horas", "minutos", "unidades", "maoMin", "extras",
                  "falhaPct": null, "margemPct": null, "precoVenda": null,   // null = usa o padrão de Config / o preço sugerido
                  "filamentos": [ { "filamentoId", "g" } ], "obs" } ],
  "vendas": [ { "id", "data", "cliente", "itens": [ { "produtoId"|null, "nome", "qtd", "preco", "custo" } ],
                "frete", "cobrado", "metodo", "custo", "taxa", "lucro", "obs", "clienteId" } ],
  "despesas": [ { "id", "data", "categoria", "descricao", "valor" } ],
  "dizimos": { "AAAA-MM": { "itens": [ { "nome", "valor" } | { "nome", "auto": true } ] } },
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

**Orçamento**: mesmos itens da venda, mas não entra em faturamento nem lucro. "✅ Aprovar" abre Vendas preenchida
(`orcEmVenda`); ao salvar a venda, o orçamento vira `aprovado` com `vendaId`. Apagar essa venda devolve o orçamento para `aberto`.
"📤 Enviar" monta um texto para WhatsApp (compartilhar do celular ou copiar).

**Clientes**: código automático em sequência (`proximoCliente`). Venda ou orçamento com nome novo cadastra o cliente sozinho
(`garantirCliente`, sem telefone) e guarda `clienteId`. Ao abrir, nomes antigos sem cadastro também entram. Trocar o nome no cadastro
atualiza vendas e orçamentos. Com telefone, "📤 Enviar" do orçamento abre a conversa no WhatsApp (`wa.me/55…`).

**Peça sob medida** (no orçamento): mesma conta de `custoProduto` (gramas e tempo do total ÷ quantidade = por peça) e `precoSugerido`.
Vira item avulso do orçamento com `detalhe` (gramas e tempo); "Salvar também em Produtos" cria o produto com categoria "Sob medida".

**Parcelamentos** (aba Despesas): parcela = total ÷ parcelas; parte = parcela ÷ pessoas; ✓ por mês e pessoa em `pagos`.
**Não** entram como despesa: o valor da impressora já entra no custo das peças (desgaste). Cadastrado: Bambu Lab A1, R$ 4.499,10,
10x a partir de dez/2026, 3 pessoas (R$ 449,91 por mês, R$ 149,97 cada).

**Despesas**: compra de filamento **não** é despesa (já entra no custo de cada venda). Fixas copiáveis: MEI, Aluguel, Internet.
**Dízimo e Resumo**: mesmas regras da Oficina; o item automático se chama "Impressão 3D" (= lucro − despesas do mês).

## Boas-vindas e novidades (desde 1.1.0)

- **Primeiro acesso** em cada aparelho: janela com o passo a passo (`PASSOS`): custos em Config → Produtos → Vendas → Despesas → Geral/Resumo.
- **Depois de cada atualização**: janela com as entradas de `NOVIDADES` mais novas que a última versão vista no aparelho.
- Guardado só no aparelho: `p3d.passosVistos` e `p3d.versaoVista`. Rever a qualquer hora: Config › Ajuda, ou tocando na versão no rodapé.

## Abas

Geral (KPIs, gráfico mensal, produtos mais vendidos, formas de pagamento, mapa mês × ano, fechamento) ·
**Orçamentos** (proposta, peça sob medida, enviar, aprovar → venda) · Vendas (itens, pagamento em botões, lista do mês, Excel) · **Produtos** (calculadora + catálogo) ·
Despesas (e parcelamentos) · Clientes (cadastro com código) · Dízimo · Resumo · Config (custos de produção, impressoras, filamentos, taxas, ajuda, exportar).

**Valores usados em Config (2026-11-07)**: energia R$ 1,00/kWh (EDP São Paulo, que atende Mogi das Cruzes: ~R$ 0,80 sem impostos,
~R$ 0,95–1,00 com ICMS, PIS/COFINS e bandeira; o certo é conta de luz ÷ kWh). Bambu Lab A1: 95 W médios em PLA (wiki da Bambu),
R$ 4.499,10 (preço total pago, 1.3.0), vida útil 5.000 h, manutenção R$ 0,20/h (bico, placa PEI, correias). Cópias de segurança do `dados.json` ficam em `backup/` (fora do git).

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
| 1.0.3 | 2026-11-05 | Manifest sem o campo `id` (igual à Oficina): a troca de identidade do app deixava o Android achar que ele já estava instalado e não abrir. Não colocar `id` de volta, nem mudar `start_url`, sem desinstalar o app dos aparelhos |
| 1.0.4 | 2026-11-05 | Endereço novo: repositório renomeado para `impressao-3d` (site `/impressao-3d/painel.html`), para o Android instalar o app do zero, sem o registro quebrado do endereço antigo |
| 1.1.0 | 2026-11-06 | Passo a passo no primeiro acesso e janela de novidades a cada versão (o que mudou + como usar); Config › Ajuda e versão no rodapé abrem as duas. Regra: toda versão ganha entrada em `NOVIDADES` |
| 1.2.0 | 2026-11-07 | Aba Orçamentos (proposta, enviar pelo WhatsApp, aprovar → venda preenchida); excluir volta a funcionar (confirmação dentro do painel no lugar de `confirm()`); saem a venda por voz (Gemini) e o tamanho da letra; Config com Bambu Lab A1 e energia da EDP |
| 1.3.0 | 2026-11-07 | Cadastro de clientes com código automático e telefone (WhatsApp); parcelamentos com ✓ por mês e pessoa (A1 em 10x dividida em 3); peça sob medida no orçamento pelos materiais; Enviar abre o WhatsApp do cliente |

# Painel Impressão 3D

Painel para calcular custo e preço das peças impressas, lançar vendas, despesas e dízimo, com histórico e gráficos.
Os dados ficam no arquivo `dados.json` (Google Drive › Meu Drive › Impressora 3D), que **não** vai para o GitHub.

## Como abrir

- **Computador (Chrome/Edge):** abrir `painel.html` direto de `H:\Meu Drive\Impressora 3D` e escolher o `dados.json` na primeira vez.
- **Tablet/celular:** abrir o endereço do GitHub Pages, tocar em "Abrir pelo Google Drive" e escolher o `dados.json` na primeira vez.

## Primeiros passos

1. **Config › Custos de produção:** preço da energia (R$/kWh), valor da sua hora de trabalho, % de perdas e margem padrão.
2. **Config › Impressoras:** potência média, quanto pagou e vida útil estimada.
3. **Config › Materiais e estoque:** cada filamento com o preço por kg e o que você tem em casa (e as compras de rolos).
4. **Produtos:** para cada peça, o tempo e os gramas que o fatiador mostra. O painel calcula custo e preço.
5. **Orçamentos:** monte a proposta (do catálogo ou peça sob medida), envie pelo WhatsApp e aprove para virar venda.
6. **Vendas:** escolha os produtos, a quantidade e a forma de pagamento.

O próprio painel mostra um passo a passo no primeiro acesso e as novidades depois de cada atualização (Config › Ajuda).

## Para continuar o desenvolvimento (em qualquer máquina ou IA)

Ler `MEMORIA.md` (tudo sobre o projeto, estado atual e histórico) e `AGENTS.md` (regras para IAs).
O `CLAUDE.md` e o `GEMINI.md` só apontam para esses dois.

## Configuração do Google (feita uma vez, na conta do projeto)

O modo Google Drive precisa de três valores em `painel.html` (constante `GOOGLE`):

| Campo      | Onde pegar no Google Cloud Console                                   |
|------------|----------------------------------------------------------------------|
| `clientId` | APIs e serviços › Credenciais › ID do cliente OAuth (Aplicativo da Web) |
| `apiKey`   | APIs e serviços › Credenciais › Chave de API (restrita à Google Picker API) |
| `appId`    | Página inicial do projeto › "Número do projeto"                        |

No projeto: ativar **Google Drive API** e **Google Picker API**; tela de consentimento OAuth "Externo",
com o e-mail da conta do Drive como usuário de teste; no ID do cliente, em "Origens JavaScript autorizadas",
colocar o endereço do GitHub Pages e `http://localhost:8765` (para testes).
Esses valores não são segredos: o painel só enxerga o arquivo escolhido no seletor (permissão `drive.file`).

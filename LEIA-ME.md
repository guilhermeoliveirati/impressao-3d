# Painel Impressão 3D

Painel para calcular custo e preço das peças impressas, lançar vendas, despesas e dízimo, com histórico e gráficos.
Os dados ficam no arquivo `dados.json` (Google Drive › Meu Drive › Impressora 3D), que **não** vai para o GitHub.

## Como abrir

- **Computador (Chrome/Edge):** abrir `painel.html` direto de `H:Meu DriveImpressora 3D` e escolher o `dados.json` na primeira vez.
- **Tablet/celular:** abrir o endereço do GitHub Pages, tocar em "Abrir pelo Google Drive" e escolher o `dados.json` na primeira vez.

## Primeiros passos

1. **Config › Custos de produção:** preço da energia (R$/kWh), valor da sua hora de trabalho, % de perdas e margem padrão.
2. **Config › Impressoras:** potência média, quanto pagou e vida útil estimada.
3. **Config › Filamentos:** cada cor/material com o preço por kg.
4. **Produtos:** para cada peça, o tempo e os gramas que o fatiador mostra. O painel calcula custo e preço.
5. **Vendas:** escolha os produtos, a quantidade e a forma de pagamento.

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

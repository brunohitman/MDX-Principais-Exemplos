# MDX-Principais-Exemplos

Repositório com principais exemplos para MDX

# Seleção de Elementos:

## a) Exibição de Tuplas:
Para exibir os valores de uma medida para combinações específicas de elementos de dimensões diferentes. É útil para criar tabelas e matrizes de dados.
Exemplo:



SELECT
  {[Product].[Category].[Electronics], [Product].[Subcategory].[Phones]} ON COLUMNS,
  {[Measures].[SalesAmount]} ON ROWS
FROM [CubeName]

## b) Cláusula Members:
Para exibir todos os membros de uma hierarquia. Útil para listar todos os elementos disponíveis em uma dimensão.
Exemplo:



SELECT
  {[Customer].[City].Members} ON COLUMNS,
  {[Measures].[Revenue]} ON ROWS
FROM [CubeName]

## c) Navegando na Hierarquia:
Para navegar pela hierarquia de uma dimensão e selecionar elementos específicos em diferentes níveis.
Exemplo:



SELECT
  {[Date].[Year].[2022].[Q2].[April], [Date].[Year].[2023].[Q1].[January]} ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]

## d) Excluir Vazios da Exibição:
Para filtrar os resultados e excluir linhas ou colunas onde a medida não possui valor (NULL).
Exemplo:



SELECT
  {[Product].[Category].[Category].Members} ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
WHERE ([Measures].[Sales] <> NULL)

# Cálculos entre Elementos:

## a) Cálculo WITH:
Para criar novas medidas com base em cálculos envolvendo medidas existentes ou membros.
Exemplo:



WITH
  MEMBER [Measures].[ProfitMargin] AS [Measures].[Profit] / [Measures].[Revenue]
SELECT
  {[Product].[Category].[Electronics], [Product].[Subcategory].[Phones]} ON COLUMNS,
  {[Measures].[ProfitMargin]} ON ROWS
FROM [CubeName]

## b) Criar Novas Medidas:
Para criar novas medidas calculadas que não estão presentes no cubo, mas podem ser derivadas de medidas existentes.
Exemplo:



CREATE MEMBER [Measures].[GrowthRate] AS
  ([Measures].[SalesThisYear] - [Measures].[SalesLastYear]) / [Measures].[SalesLastYear], FORMAT_STRING = 'Percent'
SELECT
  {[Product].[Category].[Category].Members} ON COLUMNS,
  {[Measures].[GrowthRate]} ON ROWS
FROM [CubeName]

## c) Named Set:
Para criar conjuntos de membros que podem ser reutilizados em várias consultas.
Exemplo:



CREATE SET [Top5Customers] AS
  TopCount([Customer].[Customer].[Customer].Members, 5, [Measures].[Sales])
SELECT
  {[Top5Customers]} ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]

# Ordem de Exibição de Elementos:
## a) Crossjoin:
Para criar uma combinação de membros de diferentes hierarquias, criando uma tabela de todas as possíveis combinações.
Exemplo:



SELECT
  Crossjoin([Product].[Category].[Category].Members, [Date].[Month].[Month].Members) ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]

## b) Hierarchize:
Para organizar os membros em uma hierarquia específica.
Exemplo:



SELECT
  Hierarchize({[Product].[Category].Members, [Product].[Subcategory].Members}) ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]

## c) Order:
Para ordenar os membros de uma hierarquia com base em uma medida específica.
Exemplo:



SELECT
  {[Product].[Category].[Category].Members} ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
ORDER([Product].[Category].[Category].Members, [Measures].[Sales], BDESC)

## d) Filter:
Para filtrar membros com base em uma condição específica.
Exemplo:



SELECT
  {[Product].[Category].[Category].Members} ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
WHERE [Measures].[Sales] > 1000

# Períodos de Tempo:

## a) Períodos Subsequentes:
Para exibir valores para períodos de tempo consecutivos.
Exemplo:



SELECT
  {[Date].[Month].[Month].&[2023]&[1], [Date].[Month].[Month].&[2023]&[2], [Date].[Month].[Month].&[2023]&[3]} ON COLUMNS,
  {[Measures].[Revenue]} ON ROWS
FROM [CubeName]

## b) Navegar entre Períodos:
Para exibir valores para períodos de tempo anteriores ou posteriores.
Exemplo:



SELECT
  {[Date].[Month].[Month].&[2023]&[7].PrevMember, [Date].[Month].[Month].&[2023]&[7]} ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]

## c) Períodos Paralelos:
Para exibir valores para períodos de tempo correspondentes em anos diferentes.
Exemplo:



SELECT
  {[Date].[Month].[Month].&[2022], [Date].[Month].[Month].&[2023]} ON COLUMNS,
  {[Measures].[Profit]} ON ROWS
FROM [CubeName]

## d) Períodos até a Data:
Para exibir valores acumulados até uma determinada data.
Exemplo:



SELECT
  PeriodsToDate([Date].[Month].[Year].&[2023], [Date].[Month].[Month].&[2023]&[7]) ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]

## e) Deslocamento na Dimensão:
Para exibir valores deslocados na dimensão do tempo.
Exemplo:



SELECT
  {[Date].[Month].[Month].&[2023]&[7].Lag(1)} ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]

# Ordenação de Dados:

## Top:
O Top é usado para exibir os melhores resultados com base em uma medida específica. É útil quando você deseja mostrar os principais elementos com os maiores valores de uma determinada métrica.
Exemplo:


SELECT
  {[Product].[Category].[Category].Members} ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
ORDER([Product].[Category].[Category].Members, [Measures].[Sales], BDESC)
FETCH FIRST 5 ROWS ONLY

Neste exemplo, estamos selecionando as categorias de produtos e ordenando-as com base nas vendas em ordem descendente (BDESC). Em seguida, usamos a cláusula FETCH FIRST 5 ROWS ONLY para mostrar apenas as 5 principais categorias em termos de vendas.

Bottom:
O Bottom é usado para exibir os piores resultados com base em uma medida específica. É útil quando você deseja mostrar os elementos com os menores valores de uma determinada métrica.
Exemplo:


SELECT
  {[Product].[Category].[Category].Members} ON COLUMNS,
  {[Measures].[Profit]} ON ROWS
FROM [CubeName]
ORDER([Product].[Category].[Category].Members, [Measures].[Profit], BASC)
FETCH FIRST 3 ROWS ONLY

Neste exemplo, estamos selecionando as categorias de produtos e ordenando-as com base nos lucros em ordem ascendente (BASC). Usamos a cláusula FETCH FIRST 3 ROWS ONLY para mostrar apenas as 3 piores categorias em termos de lucro.

## Rank:
A função Rank é usada para atribuir um ranking com base em uma medida específica. Ela classifica os membros da dimensão com base no valor da medida.
Exemplo:


WITH
  MEMBER [Measures].[RankSales] AS
    Rank([Product].[Product].CurrentMember, [Measures].[Sales])
SELECT
  {[Product].[Product].Members} ON COLUMNS,
  {[Measures].[Sales], [Measures].[RankSales]} ON ROWS
FROM [CubeName]
ORDER([Measures].[RankSales], BASC)

Neste exemplo, criamos uma nova medida chamada [RankSales], que atribui um ranking para cada produto com base nas vendas. Em seguida, ordenamos o resultado com base no ranking em ordem ascendente (BASC).

## Count:
A função Count é usada para contar o número de elementos em um conjunto. É útil para exibir a contagem de membros em uma dimensão ou hierarquia.
Exemplo:


SELECT
  {[Product].[Category].[Category].Members} ON COLUMNS,
  {[Measures].[Sales], [Measures].[Sales].COUNT} ON ROWS
FROM [CubeName]

Neste exemplo, estamos exibindo as categorias de produtos e a contagem de produtos em cada categoria (contagem de membros) junto com as vendas. A medida [Measures].[Sales].COUNT é usada para contar os membros da dimensão [Product].[Category].

# Formatação de Dados Numéricos:
## a) Exibir números com duas casas decimais:



WITH
  MEMBER [Measures].[VendasFormatadas] AS
    Format([Measures].[Vendas], '0.00')
SELECT
  {[Dimensao1].[Elemento1], [Dimensao2].[Elemento2]} ON COLUMNS,
  {[Measures].[VendasFormatadas]} ON ROWS
FROM [CubeName]

## b) Exibir números com separador de milhares:



WITH
  MEMBER [Measures].[VendasFormatadas] AS
    Format([Measures].[Vendas], '#,##0.00')
SELECT
  {[Dimensao1].[Elemento1], [Dimensao2].[Elemento2]} ON COLUMNS,
  {[Measures].[VendasFormatadas]} ON ROWS
FROM [CubeName]
Formatação de Dados Percentuais:

## c) Exibir percentuais com duas casas decimais:



WITH
  MEMBER [Measures].[TaxaDeConversaoFormatada] AS
    Format([Measures].[TaxaDeConversao], '0.00%')
SELECT
  {[Dimensao1].[Elemento1], [Dimensao2].[Elemento2]} ON COLUMNS,
  {[Measures].[TaxaDeConversaoFormatada]} ON ROWS
FROM [CubeName]

# Formatação de Dados Textuais:

## c) Exibir os nomes dos produtos em letras maiúsculas:



WITH
  MEMBER [DimensaoProduto].[Produto].[Produto].Properties('Caption') AS
    UCase([DimensaoProduto].[Produto].CurrentMember.Member_Caption)
SELECT
  {[DimensaoProduto].[Produto].[Produto].Members} ON COLUMNS,
  {[Measures].[Vendas]} ON ROWS
FROM [CubeName]

## d) Exibir o nome do cliente em letras minúsculas:



WITH
  MEMBER [DimensaoCliente].[Cliente].[Cliente].Properties('Caption') AS
    LCase([DimensaoCliente].[Cliente].CurrentMember.Member_Caption)
SELECT
  {[DimensaoCliente].[Cliente].[Cliente].Members} ON COLUMNS,
  {[Measures].[Vendas]} ON ROWS
FROM [CubeName]

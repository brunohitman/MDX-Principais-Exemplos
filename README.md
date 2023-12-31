# MDX-Principais-Exemplos

Repositório com principais exemplos para MDX

# Seleção de Elementos:

## a) Exibição de Tuplas:
Para exibir os valores de uma medida para combinações específicas de elementos de dimensões diferentes. É útil para criar tabelas e matrizes de dados.
Exemplo:
```ruby
SELECT
    {[Product].[Category].[Electronics], [Product].[Subcategory].[Phones]} ON COLUMNS,
    {[Measures].[SalesAmount]} ON ROWS
FROM [CubeName]
```
## b) Cláusula Members:
Para exibir todos os membros de uma hierarquia. Útil para listar todos os elementos disponíveis em uma dimensão.
Exemplo:


```ruby
SELECT
  {[Customer].[City].Members} ON COLUMNS,
  {[Measures].[Revenue]} ON ROWS
FROM [CubeName]
```
## c) Navegando na Hierarquia:
Para navegar pela hierarquia de uma dimensão e selecionar elementos específicos em diferentes níveis.
Exemplo:


```ruby
SELECT
  {[Date].[Year].[2022].[Q2].[April], [Date].[Year].[2023].[Q1].[January]} ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
```
## d) Excluir Vazios da Exibição:
Para filtrar os resultados e excluir linhas ou colunas onde a medida não possui valor (NULL).
Exemplo:


```ruby
SELECT
  {[Product].[Category].[Category].Members} ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
WHERE ([Measures].[Sales] <> NULL)
```
# Cálculos entre Elementos:

## a) Cálculo WITH:
Para criar novas medidas com base em cálculos envolvendo medidas existentes ou membros.
Exemplo:


```ruby
WITH
  MEMBER [Measures].[ProfitMargin] AS [Measures].[Profit] / [Measures].[Revenue]
SELECT
  {[Product].[Category].[Electronics], [Product].[Subcategory].[Phones]} ON COLUMNS,
  {[Measures].[ProfitMargin]} ON ROWS
FROM [CubeName]
```
## b) Criar Novas Medidas:
Para criar novas medidas calculadas que não estão presentes no cubo, mas podem ser derivadas de medidas existentes.
Exemplo:


```ruby
CREATE MEMBER [Measures].[GrowthRate] AS
  ([Measures].[SalesThisYear] - [Measures].[SalesLastYear]) / [Measures].[SalesLastYear], FORMAT_STRING = 'Percent'
SELECT
  {[Product].[Category].[Category].Members} ON COLUMNS,
  {[Measures].[GrowthRate]} ON ROWS
FROM [CubeName]
```
## c) Named Set:
Para criar conjuntos de membros que podem ser reutilizados em várias consultas.
Exemplo:


```ruby
CREATE SET [Top5Customers] AS
  TopCount([Customer].[Customer].[Customer].Members, 5, [Measures].[Sales])
SELECT
  {[Top5Customers]} ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
```
# Ordem de Exibição de Elementos:
## a) Crossjoin:
Para criar uma combinação de membros de diferentes hierarquias, criando uma tabela de todas as possíveis combinações.
Exemplo:


```ruby
SELECT
  Crossjoin([Product].[Category].[Category].Members, [Date].[Month].[Month].Members) ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
```
## b) Hierarchize:
Para organizar os membros em uma hierarquia específica.
Exemplo:


```ruby
SELECT
  Hierarchize({[Product].[Category].Members, [Product].[Subcategory].Members}) ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
```
## c) Order:
Para ordenar os membros de uma hierarquia com base em uma medida específica.
Exemplo:


```ruby
SELECT
  {[Product].[Category].[Category].Members} ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
ORDER([Product].[Category].[Category].Members, [Measures].[Sales], BDESC)
```
## d) Filter:
Para filtrar membros com base em uma condição específica.
Exemplo:


```ruby
SELECT
  {[Product].[Category].[Category].Members} ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
WHERE [Measures].[Sales] > 1000
```
# Períodos de Tempo:

## a) Períodos Subsequentes:
Para exibir valores para períodos de tempo consecutivos.
Exemplo:


```ruby
SELECT
  {[Date].[Month].[Month].&[2023]&[1], [Date].[Month].[Month].&[2023]&[2], [Date].[Month].[Month].&[2023]&[3]} ON COLUMNS,
  {[Measures].[Revenue]} ON ROWS
FROM [CubeName]
```
## b) Navegar entre Períodos:
Para exibir valores para períodos de tempo anteriores ou posteriores.
Exemplo:


```ruby
SELECT
  {[Date].[Month].[Month].&[2023]&[7].PrevMember, [Date].[Month].[Month].&[2023]&[7]} ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
```
## c) Períodos Paralelos:
Para exibir valores para períodos de tempo correspondentes em anos diferentes.
Exemplo:


```ruby
SELECT
  {[Date].[Month].[Month].&[2022], [Date].[Month].[Month].&[2023]} ON COLUMNS,
  {[Measures].[Profit]} ON ROWS
FROM [CubeName]
```
## d) Períodos até a Data:
Para exibir valores acumulados até uma determinada data.
Exemplo:


```ruby
SELECT
  PeriodsToDate([Date].[Month].[Year].&[2023], [Date].[Month].[Month].&[2023]&[7]) ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
```
## e) Deslocamento na Dimensão:
Para exibir valores deslocados na dimensão do tempo.
Exemplo:


```ruby
SELECT
  {[Date].[Month].[Month].&[2023]&[7].Lag(1)} ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
```
# Ordenação de Dados:

## a) Exibir os Top 5 produtos com maiores vendas:

```ruby
SELECT
  TopCount([Product].[Product].[Product].Members, 5, [Measures].[Sales]) ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
```

## b) Exibir os Bottom 3 produtos com menores vendas:

```ruby
SELECT
  BottomCount([Product].[Product].[Product].Members, 3, [Measures].[Sales]) ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
```
## Rank:
A função Rank é usada para atribuir um ranking com base em uma medida específica. Ela classifica os membros da dimensão com base no valor da medida.
Exemplo:

```ruby
WITH
  MEMBER [Measures].[RankSales] AS
    Rank([Product].[Product].CurrentMember, [Measures].[Sales])
SELECT
  {[Product].[Product].Members} ON COLUMNS,
  {[Measures].[Sales], [Measures].[RankSales]} ON ROWS
FROM [CubeName]
ORDER([Measures].[RankSales], BASC)
```
Neste exemplo, criamos uma nova medida chamada [RankSales], que atribui um ranking para cada produto com base nas vendas. Em seguida, ordenamos o resultado com base no ranking em ordem ascendente (BASC).

## Count:
A função Count é usada para contar o número de elementos em um conjunto. É útil para exibir a contagem de membros em uma dimensão ou hierarquia.
Exemplo:
```ruby
SELECT
  {[Product].[Category].[Category].Members} ON COLUMNS,
  {[Measures].[Sales], [Measures].[Sales].COUNT} ON ROWS
FROM [CubeName]
```
Neste exemplo, estamos exibindo as categorias de produtos e a contagem de produtos em cada categoria (contagem de membros) junto com as vendas. A medida [Measures].[Sales].COUNT é usada para contar os membros da dimensão [Product].[Category].

## Head:
Exibir os primeiros 3 produtos com maiores vendas:
```ruby
SELECT
  Head(Order([Product].[Product].[Product].Members, [Measures].[Sales], BDESC), 3) ON COLUMNS,
  {[Measures].[Sales]} ON ROWS
FROM [CubeName]
```

Neste exemplo, a função Order é utilizada para ordenar os produtos em ordem descendente com base nas vendas. Em seguida, a função Head é aplicada para retornar os três primeiros produtos com maiores vendas.

Essa combinação de Order e Head é útil quando você deseja visualizar os primeiros elementos em um conjunto ordenado, como os principais produtos por vendas ou os principais clientes por lucro.

# Formatação de Dados Numéricos:
## a) Exibir números com duas casas decimais:


```ruby
WITH
  MEMBER [Measures].[VendasFormatadas] AS
    Format([Measures].[Vendas], '0.00')
SELECT
  {[Dimensao1].[Elemento1], [Dimensao2].[Elemento2]} ON COLUMNS,
  {[Measures].[VendasFormatadas]} ON ROWS
FROM [CubeName]
```
## b) Exibir números com separador de milhares:


```ruby
WITH
  MEMBER [Measures].[VendasFormatadas] AS
    Format([Measures].[Vendas], '#,##0.00')
SELECT
  {[Dimensao1].[Elemento1], [Dimensao2].[Elemento2]} ON COLUMNS,
  {[Measures].[VendasFormatadas]} ON ROWS
FROM [CubeName]
Formatação de Dados Percentuais:
```
## c) Exibir percentuais com duas casas decimais:


```ruby
WITH
  MEMBER [Measures].[TaxaDeConversaoFormatada] AS
    Format([Measures].[TaxaDeConversao], '0.00%')
SELECT
  {[Dimensao1].[Elemento1], [Dimensao2].[Elemento2]} ON COLUMNS,
  {[Measures].[TaxaDeConversaoFormatada]} ON ROWS
FROM [CubeName]
```
# Formatação de Dados Textuais:

## c) Exibir os nomes dos produtos em letras maiúsculas:


```ruby
WITH
  MEMBER [DimensaoProduto].[Produto].[Produto].Properties('Caption') AS
    UCase([DimensaoProduto].[Produto].CurrentMember.Member_Caption)
SELECT
  {[DimensaoProduto].[Produto].[Produto].Members} ON COLUMNS,
  {[Measures].[Vendas]} ON ROWS
FROM [CubeName]
```
## d) Exibir o nome do cliente em letras minúsculas:


```ruby
WITH
  MEMBER [DimensaoCliente].[Cliente].[Cliente].Properties('Caption') AS
    LCase([DimensaoCliente].[Cliente].CurrentMember.Member_Caption)
SELECT
  {[DimensaoCliente].[Cliente].[Cliente].Members} ON COLUMNS,
  {[Measures].[Vendas]} ON ROWS
FROM [CubeName]
```

# EXTRA - UMA ANÁLISE DE PARETO

A análise de Pareto, também conhecida como "Princípio 80/20" ou "Regra 80/20", é um conceito que sugere que aproximadamente 80% dos efeitos vêm de 20% das causas. Essa ideia foi popularizada pelo economista italiano Vilfredo Pareto, que observou que a maioria da riqueza estava concentrada em uma minoria da população. O princípio de Pareto é amplamente aplicado em diversas áreas, incluindo negócios, economia, gerenciamento de projetos e análise de dados.

No código MDX fornecido, a análise classifica as marcas de produtos com base na margem de lucro e, em seguida, mostra as principais marcas que geram a maior parte do lucro. Através da medida de "Margem de Lucro Acumulada", é possível identificar quais marcas contribuem significativamente para o lucro total, e a medida "% Lucro Acumulado" mostra a proporção do lucro acumulado em relação ao total.

Essa é uma aplicação da análise de Pareto, pois a análise foca nas principais marcas (os 20%) que representam a maior parte do lucro (os 80%). Essa abordagem ajuda os gestores a identificar as áreas mais impactantes nos resultados financeiros e concentrar seus esforços nas marcas de maior relevância para otimizar o desempenho global do negócio.
```ruby
WITH
  SET [MARCAS] AS
    ORDER([Produto].[Hierarquia de Produtos].[Nível Marca], [Measures].[Margem], BDESC)
MEMBER [Measures].[Margem por Marca] AS
    '[Measures].[Margem]', FORMAT_STRING = "##,###.00"
MEMBER [Measures].[Posição no Rank] AS
    'RANK([Produto].[Hierarquia de Produtos].CurrentMember, [MARCAS])', FORMAT_STRING = "#;#;-"
MEMBER [Measures].[% Lucro] AS
    '[Measures].[Margem]/([Measures].[Margem], [Produto].[Hierarquia de Produtos].[All])', FORMAT_STRING = "#,###.00 %"
MEMBER [Measures].[Margem Acumulada] AS
    'SUM(HEAD([MARCAS],[Measures].[Posição no Rank]),[Measures].[Margem])', FORMAT_STRING = "#,###.00"
MEMBER [Measures].[Total de Produtos] AS
    '[MARCAS].Count', FORMAT_STRING = "#;#;-"
MEMBER [Measures].[% Número Produtos] AS
    '[Measures].[Posição no Rank] / [Measures].[Total de Produtos]', FORMAT_STRING = "#,###.00 %"
MEMBER [Measures].[% Lucro Acumuladas] AS
    'SUM(HEAD([MARCAS],[Measures].[Posição no Rank]),[Measures].[% Lucro] )', FORMAT_STRING = "#,###.00 %"
SELECT ({ [MARCAS] }) ON ROWS,
       ({ [Measures].[Margem por Marca]
        , [Measures].[% Lucro Acumuladas]
        , [Measures].[% Número Produtos]
        }) ON COLUMNS
FROM [COMPLETO]
WHERE ([Tempo].[Ano].&[2014])
```
O código MDX fornecido é usado para realizar uma análise de margem de lucro, classificar os produtos por marca com base na margem de lucro, calcular algumas medidas relacionadas e, em seguida, filtrar os resultados para o ano de 2014.

Vamos analisar as partes do código e sua utilidade para o usuário:

SET [MARCAS]:
Esta parte do código cria um conjunto chamado [MARCAS] que contém as marcas de produtos (nível "Marca") ordenadas em ordem descendente com base na medida [Margem]. Isso significa que as marcas serão classificadas da maior margem de lucro para a menor.

MEMBER [Measures].[Margem por Marca]:
Esta é uma medida que calcula a margem de lucro para cada marca de produto. A medida é formatada para exibir duas casas decimais.

MEMBER [Measures].[Posição no Rank]:
Esta medida utiliza a função RANK para atribuir uma posição de ranking para cada marca de produto com base na medida [Margem]. O formato da medida é definido para mostrar o número positivo caso o produto esteja no ranking e negativo caso não esteja no conjunto classificado.

MEMBER [Measures].[% Lucro]:
Esta medida calcula o percentual de lucro para cada marca em relação ao total de lucro de todos os produtos. O formato da medida é definido para mostrar o resultado em formato de porcentagem com duas casas decimais.

MEMBER [Measures].[Margem Acumulada]:
Essa medida calcula a margem de lucro acumulada para cada marca. Utiliza a função SUM para somar as margens de lucro das marcas no conjunto classificado até a posição atual do produto na classificação. O formato da medida é definido para mostrar duas casas decimais.

MEMBER [Measures].[Total de Produtos]:
Essa medida simplesmente conta a quantidade de marcas de produtos no conjunto [MARCAS]. O formato da medida é definido para mostrar o número positivo.

MEMBER [Measures].[% Número Produtos]:
Essa medida calcula o percentual da posição do produto no ranking em relação ao total de produtos no conjunto [MARCAS]. O formato da medida é definido para mostrar o resultado em formato de porcentagem com duas casas decimais.

MEMBER [Measures].[% Lucro Acumuladas]:
Essa medida calcula o percentual de lucro acumulado para cada marca. Utiliza a função SUM para somar os percentuais de lucro das marcas no conjunto classificado até a posição atual do produto na classificação. O formato da medida é definido para mostrar o resultado em formato de porcentagem com duas casas decimais.

O comando SELECT é usado para visualizar os resultados. Ele apresentará os valores de margem por marca, o percentual de lucro acumulado, e o percentual da posição no ranking para cada marca, filtrando os resultados apenas para o ano de 2014.


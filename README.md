# Análise de Vendas –  (documentação técnica)

Documento de apoio ao arquivo Power BI. Descreve a carga dos dados, o modelo de relacionamentos e as métricas criadas. A leitura dos resultados é apresentada à parte.

---

## Sobre os dados

Recebi 11 arquivos extraídos do Data Warehouse: três tabelas de vendas separadas por ano (2020, 2021 e 2022), a base de produtos com categorias e subcategorias, os clientes, os territórios, as devoluções e um calendário.

Antes de qualquer coisa abri cada arquivo para olhar as colunas e o conteúdo. Isso evitou um problema que só aparece quando você presta atenção: a tabela de Território usa a coluna `SalesTerritoryKey`, enquanto Vendas e Devoluções usam `TerritoryKey`. São o mesmo conceito com nomes diferentes, e se eu tivesse confiado no nome teria criado um relacionamento quebrado.

Um ponto importante sobre o período: os dados vão de **01/01/2020 a 30/06/2022**. Ou seja, 2022 está pela metade. Sempre que aparece comparação ano contra ano no painel, sinalizei isso, porque comparar um ano inteiro com seis meses passa uma leitura errada de queda.

## Atividade 1 – Carga e tratamento

Carreguei tudo via Power Query. Os tratamentos principais foram:

- **União das vendas.** Juntei 2020, 2021 e 2022 numa única tabela de fatos. Não faz sentido manter três tabelas com a mesma estrutura, e separadas elas só atrapalham as medidas.
- **Tipagem das datas.** `OrderDate` e `StockDate` vieram como texto. Converti para data, senão nada de tempo funciona direito.
- **Limpeza da base de clientes.** O `Customer.csv` trazia algumas linhas de lixo da exportação (registros sem chave válida). Removi essas linhas e fiquei com a base de clientes consistente.
- **Calendário.** Optei por gerar a tabela de datas, porém não tuilzei pois as datas contidas na tabela dimensão englobavam todas as datas necessárias para confeção do case

## Atividade 2 – Relacionamentos

Montei o modelo em estrela, com a tabela de vendas no centro e as demais ligadas a ela:

- Vendas → Produto (`ProductKey`)
- Produto → Subcategoria → Categoria
- Vendas → Cliente (`CustomerKey`)
- Vendas → Território (`TerritoryKey` ligado ao `SalesTerritoryKey`, conforme expliquei acima)
- Vendas → Calendário (`OrderDate`)
- Devoluções → Produto e → Território, para conseguir cruzar devolução com o que foi vendido

Esse desenho deixa os filtros fluindo numa direção só e mantém as medidas rápidas e previsíveis.

## Atividade 3 – Métricas criadas

Defini as medidas seguindo a lógica do enunciado: receita é preço vezes quantidade, e lucro é receita menos custo do produto. A partir daí montei o resto.

- **Receita** = soma de Quantidade × Preço do produto
- **Custo** = soma de Quantidade × Custo do produto
- **Lucro** = Receita − Custo
- **Margem %** = Lucro ÷ Receita
- **Pedidos** = contagem distinta de `OrderNumber`
- **Itens vendidos** = soma de `OrderQuantity`
- **Ticket médio** = Receita ÷ Pedidos
- **Itens devolvidos** = soma de `ReturnQuantity`
- **Taxa de devolução** = Itens devolvidos ÷ Itens vendidos
- **Medidas de tempo** (acumulado e comparação ano a ano) construídas sobre a tabela Calendário, com a ressalva de 2022 parcial embutida na leitura dos visuais

Conferi os totais das medidas contra a base bruta para garantir que o modelo está somando certo antes de levar para o painel.

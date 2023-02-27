# Master-Dax
**Projeto de BI focado no desenvolvimento de medidas DAX**

Finalizado mais um **Projeto de BI**, utilizando o **Power BI**, decidi compartilhar meus resultados e alguns ensinamentos obtidos.

Focado no desenvolvimento de um painel com **análise completa de vendas**, nesse projeto, podemos ver análises gerais, segmentação de contextos, cruzamento de 
informações, entre outros **_insights_** pertinentes ao objetivo.

Vale ressaltar alguns pontos primordiais, mas em vezes negligenciados no **ETL dos dados**, afim de basear as informações inferidas de maneira confiável.

- Tratamento detalhado das tabelas através no **PowerQuery**, tais como, eliminação de linhas e colunas inúteis, adequação das fontes e tipos de dados, divisão ou 
agrupamento de informações, gerenciamento de parâmetros, construção de tabela calendário (se necessário), principalmente dos conceitos de tabela Fato e Dimensão,
entre outras avaliações que tratem e limpem a base de dados.

- **Pivotiamento** cuidadoso dos relacionamentos entre as tabelas _(camada semântica)_ é extremamente importante para a confiabilidade das informações, 
o entendimento do **Schema a ser utilizado, dos contextos de ativo e inativo, cardinalidade, direção do filtro e granularidade**, farão com que os resultados obtidos
sejam fieis ao objetivo desejado.

- Importante saber o resultado que se almeja com o projeto, **OKRs**, para definir previamente os **KPIs** a serem desenvolvidos, assim delimitar um layout que 
contribua para um **_Storytelling_ eficiente e agradável**.

____________________________________________________________________________________________________________________________________________________________________

Conceitos repassados, o foco desse projeto foi a construção de um **painel gerencial de KPIs de venda**, onde o maior número de aplicações de **medidas DAX** fossem
utilizadas (nesse caso mais de 100 foram construídas) em conformidade com suas funcionalidades, interações e aplicabilidades.
Ao longo da apresentação serão exemplificados algumas das medidas mais curiosas e importantes.

![DAX](https://user-images.githubusercontent.com/116115002/221635460-acddebb2-a35f-4262-8f41-d84d484d883d.png)

____________________________________________________________________________________________________________________________________________________________________

Começo com a análise geral dos indicadores de Vendas, filtradas por ano. Avaliados Faturamento Bruto (Absoluto, anual, mensal e por categoria), Quantidade de produtos
vendidos, _Tícket_ Médio por venda, Receita Líquida (absoluta e continental), Quantidade Notas Fiscais emitidas e Indicadores por países.
Nessa etapa foram utilizadas medidas mais simples, __mas quando bem utilizadas são extremamente úteis__, com agregações, iteradoras e contagem.

- **SUM, MAX, MIN, AVERAGE, DIVIDE**
```C
Média de quantidade por ítem = 
    AVERAGE(
        fVendas[Quantidade]
    )
```
- **SUMX, MAXX, MINX, AVERAGEX**
```C
Custo Médio por ìtem = 
    AVERAGEX(
        fVendas;
        fVendas[Quantidade] * RELATED(dProduto[Custo Unitário]
        )
    )
```
_OBS: A função RELATED proporciona fazer a medida entre colunas em tabelas diferentes ganhando tempo de execução._

- **DISTINCTCOUNT, COUNTROWS**
```C
Quantidade de itens Vendidos = 
    COUNTROWS(
        fVendas
        )
```

![Análise geral](https://user-images.githubusercontent.com/116115002/221635806-791e9e8d-2b0a-4e30-9ff2-555b9e5b7fcf.JPG)

___________________________________________________________________________________________________________________________________________________________________

Na sequência, foi delimitado os KPI´s de Quantidade de Clientes, Quantidade de Vendas, Receita Líquida e Faturamento Bruto (medidas já avaliadas na análise geral)
em numeros totais e fltrados por continente. Ainda foi delimitado os países por continente com maior margem de lucro. Todos podendo ser filtrados por ano.
A segregação entre continentes e países foi realizada graças as variáveis que possibilitam a análise de linha e de filtro, medidas booleanas, tabelas virtuais e 
outros conceitos.

- **CALCULATE** (o maior curinga das medidas de contexto)
```C
América do Norte Vendas = 
    CALCULATE(  
        [05 Total Vendas Bruto];
        dLocal[Continente] = "América do Norte"
)
```
_OBS: O CALCULATE utiliza o filtro de contexto para delimitar a medida já existente_

- **VALUES E EXCEPT**
```C
Quant Clientes Não Positivados = 
        COUNTROWS(
            EXCEPT(
            VALUES(dCliente[idCliente]);
            VALUES(fVendas[Cliente]
                )            
            )
        )  
```
_OBS: Utilizei a função **COUNTROWS** para contar as linhas, filtradas pela **EXCEPT**, que por sua vez retorna apenas os valores contidos na primeira Tabela. Usando
a função **VALUES** para retornar uma tabela virtual com valores distintos. Conceito parecido com o comando SQL left Join_

- **IF E SWITCH**
```C
Visão Europa = 
        SWITCH(
            FIRSTNONBLANK('TB Medidas'[Codigo]; "BRT");
            "BRT"; [32 Europa Vendas];
            "QTC"; [33 Europa Clintes Positivados];
            "QTV"; [34 Europa Quant de Vendas];
            "LIQ"; [35 Europa Receita Líquida]
)
```
_OBS: Usei a medida Booleana **SWITCH** para filtrar em verdadeiro ou falso e retornar uma informação selecionada na segmentação de dados (o IF retorna apenas
 1 contexto). Nessa medida, utilizamos a função **FIRSTNOBLANK** para retornar o primeiro valor da coluna especificada, cuja expressão não está em branco. Dessa 
forma, quando a segmentação de dados não estiver filtrando, a função **FIRSTNOBLANK** "trava" nas informações da coluna especificada._

- **SUMMARIZE (VAR E RETURN)**
```C
Top 1 Margem de Lucro = 
    var TbAux = 
        SUMMARIZE(
            dLocal;
            dLocal[País];
            "Lucro (%)";
            [15 Margem de Lucro (%)]
        )
    Return
        MAXX(
            TbAux;
            [Lucro (%)]
        )
```        
_OBS: Criei a variável **var**, utilizando a função **SUMMARIZE** para criar um tabela virtual, baseada na informação de uma tabela existente, agrupando 
uma coluna dessa tabela, nomeando uma coluna da tabela virtual e filtrando pela expressão já existente. Dessa forma, retornamos dessa tabela virtual com **RETURN**
o maior valor da coluna especificada, utilizando a função **MAXX**._

![Análise Continental](https://user-images.githubusercontent.com/116115002/221663394-0f054bfa-b289-4f0d-ab6d-d71e24ce9180.JPG)

_____________________________________________________________________________________________________________________________________________________________________

Nessa etapa fiz uma avaliação detalhada do desempenho de cada gerente, levando em consideração os indicadores de Faturamento Bruto (valores absolutos e
percentual de representatividade em relação aos demais) por ano e geral.
Nessas análises, foram utilizadas funções de exclusão de contexto de filtro,  _ranking_ e filtragem específica.

- **ALL e ALLSELECTED**
```C
Fat Bruto Total All dVendedor = 
    CALCULATE(
        [05 Total Vendas Bruto];
        ALL(dVendedor)
)
```
_OBS: A função **ALL** retira o filtro selecionado do contexto. Com a função **ALLSELECTED**, é possível agregar um filtro externo._

- **RANKX**
```C
Ranking Gerentes = 
    var TbRankingGerentes = ALL(dVendedor[Gerente])

    return
            (RANKX(
            TbRankingGerentes;
            [05 Total Vendas Bruto]
                  )
            )
```

_OBS: Nesse calculo a variável **var** cria uma tabela virtual excluíndo filtros com **ALL** e retorna com **return** um _ranking_ da tabela virtual no contexto
da medida selecionada._

- **FILTER**
```C
Faturamento Top 1 = 
    var TbGerentes = ALL(dVendedor[Gerente])     
    
    return    
        CALCULATE(
        [05 Total Vendas Bruto];
        FILTER(TbGerentes;
        [51 Ranking Gerentes] = 1
        )
)
```

_OBS: Nessa expressão a variável **var** cria uma tabela virtual excluíndo filtros com **ALL** e retorna com **return** uma **CALCULATE** com o cálculo da expressão
a ser filtrada, utilizando o **FILTER** da tabela virtual com outra expressão. Nesse caso, para filtrar apenas a primeira ocorrência._

![Análise Gerente](https://user-images.githubusercontent.com/116115002/221681549-7ca61713-1481-45e3-b3ba-74ffd183d2a4.JPG)








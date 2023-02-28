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


![DAX](https://user-images.githubusercontent.com/116115002/221953224-55916203-d7b0-467d-b991-5c4f688675ca.png)

____________________________________________________________________________________________________________________________________________________________________

**Análise Geral**

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

![Análise geral](https://user-images.githubusercontent.com/116115002/221953326-76d96537-1489-473b-b3f0-596fa8ff5ba0.JPG)

___________________________________________________________________________________________________________________________________________________________________

**Análise Continental** 

Na sequência, foi delimitado os KPI´s de Quantidade de Clientes, Quantidade de Vendas, Receita Líquida e Faturamento Bruto (medidas já avaliadas na análise geral)
afim de obter valores totais e fltrados por **continente**. Ainda foi delimitado os países por continente com maior margem de lucro. Todos podendo serem filtrados por ano. A segregação entre continentes e países foi realizada graças as variáveis que possibilitam a análise de linha e de filtro, medidas booleanas, tabelas virtuais e 
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
         VALUES(fVendas[Cliente])            
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

![Análise Continental](https://user-images.githubusercontent.com/116115002/221953421-11deb4f2-02e3-4099-b5dc-6e3d9cf6da06.JPG)

_____________________________________________________________________________________________________________________________________________________________________

**Análise de Gerentes**

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

![Análise Gerente](https://user-images.githubusercontent.com/116115002/221953540-c2fa7173-d30a-44f4-b252-f5c8397c252f.JPG)

_____________________________________________________________________________________________________________________________________________________________________

**Análise Comparativa**

Essencial em uma análise de vendas é a comparação de indicadores em relação ao período e os objetivos a serem alcançados. Nessa parte do projeto, procurei comparar
o **Faturamento X Meta X Ano Anterior** (absoluto, anual e mensal), foi possível tambem avaliar o desempenho das unidades no contexto de Meta e entre elas. Além disso
criei um indicador que fizesse a paridade entre **Faturado e Recebido** mensalmente.
Construímos para essas análises medidas temporais, filtradas e entre relacionamentos inativos.

- **DATEADD**
```C
Faturamento Ano Anterior = 
    var VVEndas =     
        CALCULATE(
        [57 Fat. Ano Atual];
        DATEADD(dCalendario[Data];-1;YEAR
               )
        )

    return

        IF(
            ISBLANK(VVendas); 
            0;
            VVendas
        )
```
_OBS: Nesse cáculo, utilizasse a variável **var** para criar uma tabela virtual. Filtrando com **Calculate** o período a ser calculado com a expressão **DATEADD**. 
Dessa forma retorna com **return** uma medida booleada **IF**, com **ISBLANK** para evitar que algum resultado fique (Em Branco)._

- **USERELATIONSHIP**
```C
Total Faturamento bruto = 
    CALCULATE(
        [05 Total Vendas Bruto];
        USERELATIONSHIP(
            fVendas[Data de Faturamento];
            dCalendario[Data])
    ) * -1
```

_OBS: Para criarmos uma expressão entre tabelas, cujas colunas estejam em um relacionamento inativo (em virtude, na maioria dos casos, em que outro relacionamento
primordial esteja ativo), utilizasse a medida **USERELATIONSHIP** para ativar a ligação._

![Análise Comparativa](https://user-images.githubusercontent.com/116115002/221949227-ac13b4ce-e146-4d46-8870-ea26d4139df2.JPG)

____________________________________________________________________________________________________________________________________________________________________

**Análise Temporal**

Em um projeto de performance de vendas, sempre que possível, é praticamente obrigatório a avaliação do período de dados que se tem disponível, a fim de gerar
indicadores comparativos em uma jornada temporal. Foram levantadas informações de evolução de Receita Líquida a cada ano, a quantidade de vendas por ano e unidade e indicadores globais de performance incluíndo **YOY** (ano sobre ano) e **PY** (período anterior).

- **SAMEPERIODLASTYEAR**
```C
Receita Liquida PY = 
    CALCULATE(
    [14 Receita Líquida];
    SAMEPERIODLASTYEAR(dCalendario[Data])
    )
```
_OBS: A medida **SAMEPERIODLSATYEAR** apenas retorna o valor do mesmo período do ano anterior._


- **DATESYTD**
```C
Receita Líquida Acumulada Ano = 
    CALCULATE([14 Receita Líquida];
    DATESYTD(dCalendario[Data])
    )
```

_OBS: Utilizei a medida **DATESYTD** para calcular a expressão acumulada ao longo do ano em questão, reiniciando o processo a cada ano._

- **DATESBETWEEN**
```C
Receita Líquida Acumulada = 
    CALCULATE(
        [14 Receita Líquida];
        DATESBETWEEN(
            dCalendario[Data];
            BLANK();
            LASTDATE(dCalendario[Data])
        )
    )
```

_OBS: Para acumularmos os valores de uma expressão sem a barreira temporal de ano, utilizei a medida **DATESBETWEEN** como filtro da **CALCULATE**. Essa medida
solicita a data inicial e a final a ser filtrada, sendo assim, a função **BLANK()** serve para travar a primeira data e a função **LASTDATE** para relacionar
sempre a última data, mesmo que possa haver atualizações de lançamentos._

![Análise Temporal](https://user-images.githubusercontent.com/116115002/221949308-76b8cbb1-ff3e-46d6-9a03-627245897b9b.JPG)

_______________________________________________________________________________________________________________________________________________________________

**Simulador de Desempenho**

Um _plus_ nas análises de vendas, criei um simulador de desempenho. Nesse painel é possível movimentar os principais indicadores que impactam na Margem de Lucro
das vendas, reutilizando apenas as medidas já criadas.

- **KPIs utilizando SWITCH**
```C
KPI VAR Margem Lucro = 
    SWITCH(
        TRUE();
        [86 Sim Var Margem de Lucro (%)] = 0; "🟡";
        [86 Sim Var Margem de Lucro (%)] > 0; "🟢";
        [86 Sim Var Margem de Lucro (%)] < 0; "🔴"
    )
```

_OBS: Para criarmos KPI´s de desempenho, utilizei a função **SWICTH** para direcionar os indicadores._

![Simulador](https://user-images.githubusercontent.com/116115002/221956803-675dd778-bf11-41c7-9d8c-571a4070e29b.JPG)

__________________________________________________________________________________________________________________________________________________________________

**Cross Sell**

Por fim criei um conjunto de informações para basear a Venda Cruzada de cada sub categoria, a fim de definir quais sub categorias possuem um maior volume de vendas
por serem faturadas na mesma compra. Para indicar essa informações utilizei uma série de medidas já observadas nesse projeto, onde juntas, geram o resultado esperado.

- **INTERSECT**
```C
Cross Nº Vendas = 
VAR TbVendas = Values(fVendas[idNota])

VAR TbVendasAux = 
    CALCULATETABLE(
    Values(fVendas[idNota]);
    ALL(dProduto);
    USERELATIONSHIP(fVendas[Produto];'Aux Produto'[IdProduto])
    )

VAR tbInter = INTERSECT(TbVendas;TbVendasAux)

VAR vResultado = COUNTROWS(tbInter)

RETURN
 
    IF(
    SELECTEDVALUE(dProduto[Sub-Categoria]) <> SELECTEDVALUE('Aux Produto'[Sub-Categoria]);
    vResultado;
    BLANK()
    )
```
![CROSS](https://user-images.githubusercontent.com/116115002/221960446-bb4bcd16-3351-4714-b64f-eda362026011.JPG)

_OBS: Essa medida sintetiza um conjunto de funções já desenvolvidos no projeto, a fim de identificar uma condição de cruzamento de dados.
A variação **var** cria uma 1ª tabela virtual, em seguida outra tabela virtual é criada utilizando o **CALCULATE** para filtrar, retirando o contexto com **ALL**
e fazendo a ligação de colunas com relacionamento inativo, utilizando o **USERELATIONSHIP**. Com as tabelas construídas, criasse uma 3ª tabela com a função
**INTERSECT** para filtrar apenas os dados contidos em ambas as tabelas indicadas. Utilizasse assim um outra variável **var**, com a contagem de linhas usando
**COUNTROWS** para obter uma quantidade de valores. Dessa forma, retorna com **return**, a medida booleana **IF** para apresentar apenas os valores da expressão
verdadeira, caso não exista valor para a expressão, a função **BLANK()** retornará (Em Branco)._

![Cross Sell](https://user-images.githubusercontent.com/116115002/221963829-95f67e75-9205-470b-a379-c0d3d3876a4b.JPG)

Finalizamos assim o projeto de **Performance de Vendas**. Foram utilizadas uma grande quantidade de métricas, indicadores e soluções para criar uma ferramenta que
visa o embassamento na tomada de decições estratégicas para uma melhor administração comercial.
 
_https://app.powerbi.com/view?r=eyJrIjoiYzc1N2U0NDUtYzZhOC00ZDIxLTliYTAtNDliMzI1YWViYjRkIiwidCI6ImZhZDgzMjE4LTk0YzUtNDMxMi04ZWFlLWIxNzY4OGU1M2I0ZiJ9_

Segue link do painel para melhor visualização do resultado do projeto.



# Modelando-um-Dashboard-de-E-commerce-com-Power-BI-Utilizando-F-rmulas-DAX
# Desafio DIO: Modelagem Dimensional no Power BI (Flat File para Star Schema)

Este repositório contém a solução para o desafio de projeto da DIO sobre modelagem de dados e DAX, focado em Business Intelligence com o Power BI.

## 1. Objetivo do Desafio
O objetivo principal foi transformar uma única tabela "flat file" (um arquivo único com todas as informações, `Financial Sample.xlsx`) em um **Modelo Dimensional Star Schema** otimizado para análises.
Este processo é fundamental no dia a dia de um Analista de BI, pois "quebra" uma tabela lenta e desnormalizada em um modelo de alta performance, composto por uma tabela Fato (com os números) e várias tabelas de Dimensão (com o contexto).

## 2. Ferramentas e Conceitos Utilizados
* **Power BI Desktop**
* **Power Query (Editor de Consultas):** Para todo o processo de ETL (Extração, Transformação e Carga).
* **DAX (Data Analysis Expressions):** Para a criação da tabela de dimensão de calendário.
* **Modelagem de Dados:** Criação de um modelo Star Schema.

## 3. Processo de Transformação (Passo a Passo)
O trabalho foi dividido em duas grandes etapas: transformação no Power Query e modelagem com DAX.

### Etapa 1: ETL com Power Query
Dentro do Editor do Power Query, realizei as seguintes transformações para criar as tabelas de dimensão:

1.  **Backup:** A tabela original foi duplicada e mantida como `Financials_origem` (sem ser carregada no modelo, "oculta"), servindo como backup. A tabela principal foi renomeada para `F_Vendas`.
2.  **Criação de Chave Substituta (Surrogate Key):** Como o arquivo original não possuía um "ID de Produto" numérico, criei uma **Coluna Condicional** para gerar um `ID_produto` (ex: "Carretera" = 0, "Montana" = 1). Isso é uma prática essencial para otimizar a performance dos relacionamentos do modelo.
3.  **Criação das Dimensões (Referência):** A partir da tabela `F_Vendas`, usei a função **"Referenciar"** para criar as novas tabelas de dimensão. Isso garante que qualquer atualização futura na fonte de dados se propague para as dimensões.
* **`D_Produtos`:**
    * Utilizei a função **"Agrupar por"** (Group By) nas colunas `ID_produto` e `Product`.
    * Adicionei agregações para calcular as métricas pedidas no desafio: `Média de Unidades Vendidas`, `Média do valor de vendas`, `Mediana do valor de vendas`, `Valor máximo de Venda` e `Valor mínimo de Venda`.

* **`D_Produtos_Detalhes` e `D_Descontos`:**
    * Selecionei as colunas relevantes para cada dimensão (ex: `ID_produto`, `Discount Band`, `Sale Price`, etc.).
    * Utilizei a função **"Remover Duplicatas"** para garantir que cada linha fosse única, criando assim uma dimensão válida.

* **Outras Dimensões:**
    * Criei dimensões adicionais (como `D_Segmento` e `D_Pais`) usando o mesmo processo de "Escolher Colunas" e "Remover Duplicatas".

### Etapa 2: Criação da D_Calendário com DAX
A dimensão de calendário, essencial em qualquer modelo de BI, foi criada do zero usando DAX, já na tela principal do Power BI.

1.  Usei a função **`CALENDAR()`** para gerar uma tabela de datas dinâmica, buscando a data mínima e máxima da minha tabela `F_Vendas`.
2.  Usei **`ADDCOLUMNS()`** para enriquecer a tabela com colunas de análise, como `Ano`, `Mes_Num`, `Nome_Mes`, `Trimestre` e `Dia_da_Semana`.
3.  **Código DAX Utilizado:**
    ```dax
    D_Calendario = 
    ADDCOLUMNS (
        CALENDAR ( 
            MIN ( F_Vendas[Date] ), 
            MAX ( F_Vendas[Date] ) 
        ),
        "Ano", YEAR ( [Date] ),
        "Mes_Num", MONTH ( [Date] ),
        "Nome_Mes", FORMAT ( [Date], "mmmm" ),
        "Ano-Mes", FORMAT ( [Date], "yyyy-mm" ),
        "Trimestre", "T" & QUARTER ( [Date] ),
        "Dia_da_Semana", FORMAT ( [Date], "dddd" )
    )
    ```
4.  Após a criação, a `D_Calendario` foi **"Marcada como tabela de data"** no Power BI, uma etapa crucial para habilitar funções de Inteligência de Tempo.
### Etapa 3: Modelagem de Dados (Star Schema)

Na "Exibição de Modelo" do Power BI, finalizei o projeto criando os relacionamentos entre a tabela Fato e as Dimensões:
* `F_Vendas[ID_produto]` -> `D_Produtos[ID_produto]`
* `F_Vendas[ID_produto]` -> `D_Produtos_Detalhes[ID_produto]`
* `F_Vendas[ID_produto]` -> `D_Descontos[ID_produto]`
* `F_Vendas[Date]` -> `D_Calendario[Date]`
* (e assim por diante para as demais dimensões)

## 4. Resultado Final
O resultado é um modelo Star Schema limpo, otimizado e pronto para a criação de relatórios de alta performance.

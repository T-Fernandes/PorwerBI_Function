## Automação de Coleta de Dados via Web Scraping com Power Query

 #### 📌 Coletando dados de Feriados Nacionais diretamente do site da [ANBIMA](https://www.anbima.com.br/feriados/fer_nacionais/2024.asp).

**Objetivo:**
Automatizar a obtenção de informações sobre feriados nacionais brasileiros a partir do site da ANBIMA, permitindo análise e visualização dos dados em Power BI.

**Etapa 1: Definindo uma função para coleta de dados** 

Esta função é usada para automatizar a obtenção de informações sobre feriados nacionais a partir do site da ANBIMA. Ele permite que o usuário especifique o ano de interesse, e o script baixa e processa a página web correspondente, extraindo a tabela de feriados, promovendo os cabeçalhos e alterando os tipos de dados para facilitar análises futuras.
```
(Ano as text) =>

let
    Fonte = Web.BrowserContents("https://www.anbima.com.br/feriados/fer_nacionais/" & Ano & ".asp"),
    #"Tabela extraída de HTML" = Html.Table(Fonte, 
    {
        {"Column1", "TABLE.interna > * > TR > :nth-child(1)"}, 
        {"Column2", "TABLE.interna > * > TR > :nth-child(2)"}, 
        {"Column3", "TABLE.interna > * > TR > :nth-child(3)"}
    }, 
    [RowSelector="TABLE.interna > * > TR"]),
    #"Cabeçalhos Promovidos" = Table.PromoteHeaders(#"Tabela extraída de HTML", [PromoteAllScalars=true]),
    #"Tipo Alterado" = Table.TransformColumnTypes(#"Cabeçalhos Promovidos",
    {
        {"Data", type date}, 
        {"Dia da Semana", type text},
        {"Feriado", type text}
    })
in
    #"Tipo Alterado"
```
![image](https://github.com/user-attachments/assets/d0dfc9af-77c8-45b9-8e89-87bb98cf6e13)

**Etapa 2: Coleta e Consolidação de Dados**

O site da ANBIMA disponibiliza informações sobre os feriados nacionais brasileiros a partir do ano de 2001. O script abaixo automatiza o processo de coleta desses dados, gerando uma tabela consolidada com todos os feriados nacionais de 2001 até o ano atual. A função previamente criada é utilizada para extrair os dados de cada ano, que são então agregados em uma única tabela com todas as informações dos feriados ao longo dos anos. 
```
let
    AnoInicial = 2001,
    AnoFinal = Date.Year(DateTime.LocalNow()),

    // Gera uma lista de números do ano de 2001 até o ano atual
    ListaAnos = List.Numbers(AnoInicial, AnoFinal - AnoInicial + 1), 
    
    // Converte a lista em tabela com a coluna "Ano"
    Fonte = Table.TransformColumnTypes(Table.FromList(ListaAnos, Splitter.SplitByNothing(), {"Ano"}), {{"Ano", type text}}),
        
    #"Função Personalizada Invocada" = Table.AddColumn(Fonte, "Feriados", each #"Feriados nacionais Anbima"([Ano])),
    #"Feriados Expandido" = Table.ExpandTableColumn(#"Função Personalizada Invocada", "Feriados", {"Data", "Dia da Semana", "Feriado"}, {"Data", "Dia da Semana", "Feriado"}),
    #"Changed Type" = Table.TransformColumnTypes(#"Feriados Expandido",{{"Data", type date}, {"Dia da Semana", type text}, {"Feriado", type text}})
in
    #"Changed Type"
```
![image](https://github.com/user-attachments/assets/d1361f79-08fa-4d52-8de6-bdda5807e048)

**Etapa 3: Visualização dos Dados de Feriados Nacionais**  

Após consolidar os dados dos feriados nacionais, podemos criar uma visualização para entender melhor a distribuição dos feriados ao longo do ano. Abaixo está um exemplo de visualização criada no Power BI utilizando os dados coletados:
![image](https://github.com/user-attachments/assets/9d96c29d-8eca-43a9-a7b1-d754c7b741d2)

#### 📌 Coletando dados da Compensação Financeira distribuída aos municípios através do site da [ANM](https://sistemas.anm.gov.br/arrecadacao/extra/Relatorios/distribuicao_cfem_muni.aspx?ano=2022&uf=PA).

**Objetivo:**
Automatizar a coleta dos dados de Compensação Financeira pela Exploração de Recursos Minerais (CFEM) distribuída aos municípios, facilitando a análise financeira de receitas municipais ao longo dos anos.

**Etapa 1: Criação de Parâmetros**

Inicialmente, criamos dois parâmetros no Power Query para filtrar os dados pela unidade federativa (UF) e pelo município. Esses parâmetros são utilizados para gerar URLs dinâmicas que permitem acessar as informações específicas do município desejado.
```
"PA" meta [IsParameterQuery=true, Type="Text", IsParameterQueryRequired=true]
```
```
"Parauapebas" meta [IsParameterQuery=true, Type="Text", IsParameterQueryRequired=true]
```
![image](https://github.com/user-attachments/assets/f1590ca6-1d19-48b9-9dad-494495e1ba19)

**Etapa 2: Coleta e Consolidação de Dados**

O site da ANM disponibiliza informações sobre a compensação financeira dos municípios a partir de 2004. O script abaixo automatiza o processo de coleta desses dados, gerando uma tabela consolidada com o valor da compensação anual recebido pelo município desde 2004 até o último ano fechado. O script utiliza os parâmetros definidos anteriormente para filtrar os dados pela unidade federativa (UF) e pelo município. Ele gera uma lista de URLs dinâmicas para cada ano, carrega os dados correspondentes e os combina em uma única tabela com todas as compensações recebidas pelo município ao longo dos anos. 
```
let
    // Cria a lista de anos
    AnoInicial = 2004,
    AnoAtual = Date.Year(DateTime.LocalNow()),
    Anos = List.Transform(List.Numbers(AnoInicial, AnoAtual - AnoInicial), each _),

    // Gera uma lista de URLs dinâmicas
    URLs = List.Transform(Anos, each "https://sistemas.anm.gov.br/arrecadacao/extra/relatorios/distribuicao_cfem_muni.aspx?ano=" & Number.ToText(_) & "&uf=" & Text.Upper(Parameter_SIGLA_UF)),

    // Função para carregar cada tabela e adicionar coluna do ano
    ObterTabela = (ano as number, url as text) => 
        let
            Fonte = Web.BrowserContents(url),
            #"Extracted Table From Html" = Html.Table(Fonte, {{"Estado", "TABLE.tabelaRelatorio > TBODY > TR > :nth-child(1)"}, {"Jan", ":not(*)"}, {"Fev", ":not(*)"}, {"Mar", ":not(*)"}, {"Abri", "TABLE.tabelaRelatorio > TBODY > TR > :nth-child(6)"}, {"Jun", "TABLE.tabelaRelatorio > TBODY > TR > :nth-child(7)"}, {"Jul", "TABLE.tabelaRelatorio > TBODY > TR > :nth-child(8)"}, {"Ago", "TABLE.tabelaRelatorio > TBODY > TR > :nth-child(9)"}, {"Set", "TABLE.tabelaRelatorio > TBODY > TR > :nth-child(10)"}, {"Out", "TABLE.tabelaRelatorio > TBODY > TR > :nth-child(11)"}, {"Nov", "TABLE.tabelaRelatorio > TBODY > TR > :nth-child(12)"}, {"Dez", "TABLE.tabelaRelatorio > TBODY > TR > :nth-child(13)"}, {"Total", "TABLE.tabelaRelatorio > TBODY > TR > :nth-child(14)"}}, [RowSelector="TABLE.tabelaRelatorio > TBODY > TR"]),
            #"Changed Type" = Table.TransformColumnTypes(#"Extracted Table From Html",{{"Estado", type text}, {"Jan", type text}, {"Fev", type text}, {"Mar", type text}, {"Abri", type number}, {"Jun", type number}, {"Jul", type number}, {"Ago", type number}, {"Set", type number}, {"Out", type number}, {"Nov", type number}, {"Dez", type number}, {"Total", Currency.Type}}),
            #"Filtered Rows" = Table.SelectRows(#"Changed Type", each ([Estado] = Text.Upper(Parameter_MUNICIPIO))),
            #"Removed Columns" = Table.RemoveColumns(#"Filtered Rows",{"Jan", "Fev", "Mar", "Abri", "Jun", "Jul", "Ago", "Set", "Out", "Nov", "Dez"}),
            #"Added Custom" = Table.AddColumn(#"Removed Columns", "Ano", each ano),
            TabelaExtraida = #"Added Custom"
        in
            TabelaExtraida,

    // Carrega e combina as tabelas
    Tabelas = List.Transform(List.Zip({Anos, URLs}), each ObterTabela(_{0}, _{1})),
    TabelaFinal = Table.Combine(Tabelas),
    #"Capitalized Each Word" = Table.TransformColumns(TabelaFinal,{{"Estado", Text.Proper, type text}})
in
    #"Capitalized Each Word"
```
![image](https://github.com/user-attachments/assets/61b39327-a786-4453-9cc6-08d1eb73d4af)

**Etapa 3: Visualização dos Dados de Compensação Financeira** 

Após consolidar os dados de compensação financeira, podemos criar uma visualização no Power BI para entender melhor a distribuição do CFEM ao longo dos anos. Abaixo está um exemplo de visualização criada no Power BI utilizando os dados coletados.
![image](https://github.com/user-attachments/assets/9a47ebd7-50b5-482c-a632-653ddc637376)


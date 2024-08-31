## Web Scraping com Power Query

 #### 📌 Coletando dados de Feriados Nacionais diretamente do site da [ANBIMA](https://www.anbima.com.br/feriados/fer_nacionais/2024.asp).

**Etapa 1** - Esta função é usada para automatizar a obtenção de informações sobre feriados nacionais a partir do site da ANBIMA. Ele permite que o usuário especifique o ano de interesse, e o script baixa e processa a página web correspondente, extraindo a tabela de feriados, promovendo os cabeçalhos e alterando os tipos de dados para facilitar análises futuras.
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

**Etapa 2** - O site da ANBIMA disponibiliza informações sobre os feriados nacionais brasileiros a partir do ano de 2001. O script abaixo automatiza o processo de coleta desses dados, gerando uma tabela consolidada com todos os feriados nacionais de 2001 até o ano atual. A função previamente criada é utilizada para extrair os dados de cada ano, que são então agregados em uma única tabela com todas as informações dos feriados ao longo dos anos. 
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

**Etapa 3** - Após consolidar os dados dos feriados nacionais, podemos criar uma visualização para entender melhor a distribuição dos feriados ao longo do ano. Abaixo está um exemplo de visualização criada no Power BI utilizando os dados coletados:
![image](https://github.com/user-attachments/assets/9d96c29d-8eca-43a9-a7b1-d754c7b741d2)

#### 📌 Coletando dados da Compensação Financeira distribuída aos municípios através do site da [ANM](https://sistemas.anm.gov.br/arrecadacao/extra/Relatorios/distribuicao_cfem_muni.aspx?ano=2022&uf=PA).

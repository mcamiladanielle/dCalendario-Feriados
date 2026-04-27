# 📅 Calendário Inteligente no Power BI

> Guia completo para criar um calendário dinâmico com feriados fixos, móveis e dicas visuais para seus relatórios.

---

## Sumário

1. [Introdução ao Calendário Inteligente](#1-introdução-ao-calendário-inteligente)
2. [Criando a Tabela Calendário no Power Query](#2-criando-a-tabela-calendário-no-power-query)
3. [Ordenando a Coluna MêsAno](#3-ordenando-a-coluna-mêsano)
4. [Adicionando Feriados Fixos e Corporativos](#4-adicionando-feriados-fixos-e-corporativos)
5. [Calculando Feriados Móveis com Função da Páscoa](#5-calculando-feriados-móveis-com-função-da-páscoa)
6. [Gerando Tabela com Feriados Móveis](#6-gerando-tabela-com-feriados-móveis)
7. [Unindo Feriados Fixos e Móveis](#7-unindo-feriados-fixos-e-móveis)
8. [Fazendo Merge com a Tabela Calendário](#8-fazendo-merge-com-a-tabela-calendário)
9. [Dicas Visuais para o Relatório](#9-dicas-visuais-para-o-relatório)
10. [Considerações Finais](#10-considerações-finais)

---

## 1. Introdução ao Calendário Inteligente

Um calendário inteligente no Power BI permite análises dinâmicas e precisas, alinhando datas, feriados e períodos importantes para sua empresa ou projeto.

Vamos construir uma **tabela calendário robusta**, incluindo feriados fixos e móveis, e aplicar ordenações e relacionamentos para facilitar a análise.

---

## 2. Criando a Tabela Calendário no Power Query

No Power Query, crie uma tabela de datas que servirá como base para seu calendário. Use o código abaixo para gerar datas entre um intervalo definido, adicionando colunas úteis como ano, mês, nome do mês, trimestre, dia da semana, entre outras.

```powerquery
let
    DataInicial = #date(2020, 1, 1),
    DataFinal   = #date(2030, 12, 31),
    ListaDatas = List.Dates(DataInicial, Duration.Days(DataFinal - DataInicial) + 1, #duration(1,0,0,0)),
    TabelaDatas = Table.FromList(ListaDatas, Splitter.SplitByNothing(), {"Data"}),
    AddAno = Table.AddColumn(TabelaDatas, "Ano", each Date.Year([Data])),
    AddMes = Table.AddColumn(AddAno, "Mês", each Date.Month([Data])),
    AddNomeMes = Table.AddColumn(AddMes, "NomeMês", each Date.MonthName([Data])),
    AddTrimestre = Table.AddColumn(AddNomeMes, "Trimestre", each Date.QuarterOfYear([Data])),
    AddDiaSemana = Table.AddColumn(AddTrimestre, "DiaSemana", each Date.DayOfWeekName([Data])),
    AddMesAno = Table.AddColumn(AddDiaSemana, "MêsAno", each Text.Start(Date.MonthName([Data]),3) & "-" & Text.From(Date.Year([Data]))),
    AddAnoMesNum = Table.AddColumn(AddMesAno, "AnoMesNum", each Date.Year([Data]) * 100 + Date.Month([Data]))
in
    AddAnoMesNum
```

> 💡 **Dica Visual:** Use ícones ou cores para destacar colunas como `Trimestre` e `DiaSemana` no relatório para facilitar a leitura.

---

## 3. Ordenando a Coluna MêsAno

Para garantir que os meses apareçam na **ordem correta** (e não em ordem alfabética), siga estes passos:

1. No **Power BI Desktop**, vá em **Modelagem**
2. Selecione a coluna **MêsAno**
3. Clique em **Classificar por coluna** e escolha **AnoMesNum**

> 💡 **Dica:** Isso evita confusões em gráficos e tabelas que usam o campo `MêsAno`.

---

## 4. Adicionando Feriados Fixos e Corporativos

Inclua feriados fixos e datas importantes da empresa criando uma tabela no Power Query:

```powerquery
let
    FeriadosFixos = #table(
        {"Data", "Feriado"},
        {
            {#date(2026, 1, 1),  "Confraternização Universal"},
            {#date(2026, 4, 21), "Tiradentes"},
            {#date(2026, 5, 1),  "Dia do Trabalho"},
            {#date(2026, 9, 7),  "Independência do Brasil"},
            {#date(2026, 10, 12),"Nossa Senhora Aparecida"},
            {#date(2026, 11, 2), "Finados"},
            {#date(2026, 11, 15),"Proclamação da República"},
            {#date(2026, 12, 25),"Natal"},
            {#date(2026, 3, 15), "Aniversário da Empresa"},
            {#date(2026, 12, 31),"Recesso Corporativo"}
        }
    )
in
    FeriadosFixos
```

> 💡 **Dica Visual:** Utilize cores diferentes para feriados nacionais e corporativos para facilitar a identificação.

---

## 5. Calculando Feriados Móveis com Função da Páscoa

Para feriados móveis como Carnaval e Corpus Christi, crie uma função para calcular a data da Páscoa e derive os outros feriados a partir dela:

```powerquery
let
    Páscoa = (ano as number) as date =>
    let
        restoAno19             = Number.Mod(ano, 19),
        seculo                 = Number.IntegerDivide(ano, 100),
        restoSeculo            = Number.Mod(ano, 100),
        quocienteSeculo4       = Number.IntegerDivide(seculo, 4),
        restoSeculo4           = Number.Mod(seculo, 4),
        ajuste25               = Number.IntegerDivide(seculo + 8, 25),
        ajuste3                = Number.IntegerDivide(seculo - ajuste25 + 1, 3),
        epacta                 = Number.Mod(19 * restoAno19 + seculo - quocienteSeculo4 - ajuste3 + 15, 30),
        quocienteRestoSeculo4  = Number.IntegerDivide(restoSeculo, 4),
        restoRestoSeculo4      = Number.Mod(restoSeculo, 4),
        ajusteSemana           = Number.Mod(32 + 2 * restoSeculo4 + 2 * quocienteRestoSeculo4 - epacta - restoRestoSeculo4, 7),
        ajusteFinal            = Number.IntegerDivide(restoAno19 + 11 * epacta + 22 * ajusteSemana, 451),
        mes                    = Number.IntegerDivide(epacta + ajusteSemana - 7 * ajusteFinal + 114, 31),
        dia                    = ((epacta + ajusteSemana - 7 * ajusteFinal + 114) mod 31) + 1,
        dataPascoa             = #date(ano, mes, dia)
    in
        dataPascoa
in
    Páscoa
```

> 💡 **Dica:** Teste a função para vários anos para garantir que os cálculos estejam corretos.

---

## 6. Gerando Tabela com Feriados Móveis

Com a função da Páscoa, crie uma tabela que calcula os feriados móveis para um intervalo de anos:

```powerquery
let
    Anos = {2020..2030},
    FeriadosMoveis     = Table.FromList(Anos, Splitter.SplitByNothing(), {"Ano"}),
    AddPascoa          = Table.AddColumn(FeriadosMoveis, "Páscoa", each Páscoa([Ano])),
    AddCarnaval        = Table.AddColumn(AddPascoa, "Carnaval", each Date.AddDays([Páscoa], -47)),
    AddSextaSanta      = Table.AddColumn(AddCarnaval, "Sexta-feira Santa", each Date.AddDays([Páscoa], -2)),
    AddCorpusChristi   = Table.AddColumn(AddSextaSanta, "Corpus Christi", each Date.AddDays([Páscoa], 60)),
    FeriadosMoveisUnpivot = Table.UnpivotOtherColumns(AddCorpusChristi, {"Ano"}, "Feriado", "Data")
in
    FeriadosMoveisUnpivot
```

> 💡 **Dica Visual:** Use gráficos de linha para mostrar a variação das datas dos feriados móveis ao longo dos anos.

---

## 7. Unindo Feriados Fixos e Móveis

Combine as tabelas de feriados fixos e móveis para ter uma visão completa:

```powerquery
let
    TodosFeriados = Table.Combine({FeriadosFixos, FeriadosMoveisUnpivot}),
    AddFlag = Table.AddColumn(TodosFeriados, "ÉFeriado", each true)
in
    AddFlag
```

---

## 8. Fazendo Merge com a Tabela Calendário

No Power Query, faça o **merge** da tabela calendário com a tabela de feriados pelo campo `Data`. Expanda as colunas de feriado e substitua valores nulos:

```powerquery
CalendarioComFeriados = Table.ReplaceValue(
    CalendarioExpandido,
    null,
    false,
    Replacer.ReplaceValue,
    {"ÉFeriado"}
)
```

> 💡 **Dica:** Crie uma coluna condicional para destacar feriados no seu relatório, usando cores ou ícones.

---

## 9. Dicas Visuais para o Relatório

- 🎨 **Cores:** Diferencie fins de semana, feriados e dias úteis com paletas distintas
- 🔖 **Ícones:** Insira ícones para eventos especiais ou datas importantes
- 🔍 **Segmentações:** Filtre por ano, trimestre ou mês com slicers
- 📈 **Gráficos:** Use gráficos de linha ou barras para analisar tendências ao longo do tempo

---

## 10. Considerações Finais

Com este guia, você terá um **calendário inteligente completo**, que pode ser adaptado para diversas análises no Power BI — desde produtividade até vendas e atendimento.

A personalização visual ajuda a tornar os relatórios mais intuitivos e impactantes.

> ✅ **Explore, teste e adapte conforme a necessidade do seu negócio!**

---

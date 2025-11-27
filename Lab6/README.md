# Исследование вредоносной активности в домене Windows
Mary

## Цель работы

1.  Закрепить навыки исследования данных журнала Windows Active
    Directory
2.  Изучить структуру журнала системы Windows Active Directory
3.  Закрепить практические навыки использования языка программирования R
    для обработки данных
4.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R

## Исходные данные

1.  Программное обеспечение Windows 11 Pro
2.  Visual Studio Code с установленными плагинами для работы с языком R
3.  Интерпретатор языка R 4.5.2

## План

1.  Распаковать архив и загрузить исходный JSON-файл с логами.
2.  Проверить структуру данных и определить вложенные поля.
3.  Раскрыть вложенные датафреймы с помощью unnest_wider, сохранив имена
    колонок.
4.  Удалить неинформативные колонки, содержащие только одно уникальное
    значение.
5.  Подсчитать количество уникальных хостов в датасете.
6.  Загрузить таблицу Windows Event_ID и привести типы данных к
    корректным.
7.  Объединить данные или использовать справочник для интерпретации
    событий.
8.  Определить число событий с высоким и средним уровнем значимости.

## Шаги:

1.  Импорт библиотек

    ::: {.cell}

    ``` r
    library(rvest)
    library(tidyverse)
    ```

    ::: {.cell-output .cell-output-stderr}

        ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
        ✔ dplyr     1.1.4     ✔ readr     2.1.6
        ✔ forcats   1.0.1     ✔ stringr   1.6.0
        ✔ ggplot2   4.0.1     ✔ tibble    3.3.0
        ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
        ✔ purrr     1.2.0     
        ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
        ✖ dplyr::filter()         masks stats::filter()
        ✖ readr::guess_encoding() masks rvest::guess_encoding()
        ✖ dplyr::lag()            masks stats::lag()
        ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

    :::

    ``` r
    library(lubridate)
    options(stringsAsFactors = FALSE)
    ```

    :::

2.  Импорт данных

    ::: {.cell}

    ``` r
    file_path <- "dataset.tar.gz"

    untar(file_path, exdir = "data")
    json_file <- list.files("data", pattern = ".json", full.names = TRUE)

    siem_data <- jsonlite::stream_in(file(json_file))
    ```

    ::: {.cell-output .cell-output-stderr}

        opening file input connection.

    :::

    ::: {.cell-output .cell-output-stdout}

    \`\`\`

Found 500 records… Found 1000 records… Found 1500 records… Found 2000
records… Found 2500 records… Found 3000 records… Found 3500 records…
Found 4000 records… Found 4500 records… Found 5000 records… Found 5500
records… Found 6000 records… Found 6500 records… Found 7000 records…
Found 7500 records… Found 8000 records… Found 8500 records… Found 9000
records… Found 9500 records… Found 10000 records… Found 10500 records…
Found 11000 records… Found 11500 records… Found 12000 records… Found
12500 records… Found 13000 records… Found 13500 records… Found 14000
records… Found 14500 records… Found 15000 records… Found 15500 records…
Found 16000 records… Found 16500 records… Found 17000 records… Found
17500 records… Found 18000 records… Found 18500 records… Found 19000
records… Found 19500 records… Found 20000 records… Found 20500 records…
Found 21000 records… Found 21500 records… Found 22000 records… Found
22500 records… Found 23000 records… Found 23500 records… Found 24000
records… Found 24500 records… Found 25000 records… Found 25500 records…
Found 26000 records… Found 26500 records… Found 27000 records… Found
27500 records… Found 28000 records… Found 28500 records… Found 29000
records… Found 29500 records… Found 30000 records… Found 30500 records…
Found 31000 records… Found 31500 records… Found 32000 records… Found
32500 records… Found 33000 records… Found 33500 records… Found 34000
records… Found 34500 records… Found 35000 records… Found 35500 records…
Found 36000 records… Found 36500 records… Found 37000 records… Found
37500 records… Found 38000 records… Found 38500 records… Found 39000
records… Found 39500 records… Found 40000 records… Found 40500 records…
Found 41000 records… Found 41500 records… Found 42000 records… Found
42500 records… Found 43000 records… Found 43500 records… Found 44000
records… Found 44500 records… Found 45000 records… Found 45500 records…
Found 46000 records… Found 46500 records… Found 47000 records… Found
47500 records… Found 48000 records… Found 48500 records… Found 49000
records… Found 49500 records… Found 50000 records… Found 50500 records…
Found 51000 records… Found 51500 records… Found 52000 records… Found
52500 records… Found 53000 records… Found 53500 records… Found 54000
records… Found 54500 records… Found 55000 records… Found 55500 records…
Found 56000 records… Found 56500 records… Found 57000 records… Found
57500 records… Found 58000 records… Found 58500 records… Found 59000
records… Found 59500 records… Found 60000 records… Found 60500 records…
Found 61000 records… Found 61500 records… Found 62000 records… Found
62500 records… Found 63000 records… Found 63500 records… Found 64000
records… Found 64500 records… Found 65000 records… Found 65500 records…
Found 66000 records… Found 66500 records… Found 67000 records… Found
67500 records… Found 68000 records… Found 68500 records… Found 69000
records… Found 69500 records… Found 70000 records… Found 70500 records…
Found 71000 records… Found 71500 records… Found 72000 records… Found
72500 records… Found 73000 records… Found 73500 records… Found 74000
records… Found 74500 records… Found 75000 records… Found 75500 records…
Found 76000 records… Found 76500 records… Found 77000 records… Found
77500 records… Found 78000 records… Found 78500 records… Found 79000
records… Found 79500 records… Found 80000 records… Found 80500 records…
Found 81000 records… Found 81500 records… Found 82000 records… Found
82500 records… Found 83000 records… Found 83500 records… Found 84000
records… Found 84500 records… Found 85000 records… Found 85500 records…
Found 86000 records… Found 86500 records… Found 87000 records… Found
87500 records… Found 88000 records… Found 88500 records… Found 89000
records… Found 89500 records… Found 90000 records… Found 90500 records…
Found 91000 records… Found 91500 records… Found 92000 records… Found
92500 records… Found 93000 records… Found 93500 records… Found 94000
records… Found 94500 records… Found 95000 records… Found 95500 records…
Found 96000 records… Found 96500 records… Found 97000 records… Found
97500 records… Found 98000 records… Found 98500 records… Found 99000
records… Found 99500 records… Found 1e+05 records… Found 100500 records…
Found 101000 records… Found 101500 records… Found 101904 records…
Imported 101904 records. Simplifying… \`\`\`

    :::

    ::: {.cell-output .cell-output-stderr}

    ```
    closing file input connection.
    ```


    :::
    :::

1.  Подготовка справочника событий Windows

    ::: {.cell}

    ``` r
    webpage <- xml2::read_html("https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/appendix-l--events-to-monitor")
    event_df <- rvest::html_table(webpage)[[1]]

    glimpse(event_df)
    ```

    ::: {.cell-output .cell-output-stdout}

        Rows: 381
        Columns: 4
        $ `Current Windows Event ID` <chr> "4618", "4649", "4719", "4765", "4766", "47…
        $ `Legacy Windows Event ID`  <chr> "N/A", "N/A", "612", "N/A", "N/A", "N/A", "…
        $ `Potential Criticality`    <chr> "High", "High", "High", "High", "High", "Hi…
        $ `Event Summary`            <chr> "A monitored security event pattern has occ…

    ::: :::

2.  Предварительный просмотр данных

    ::: {.cell}

    ``` r
    glimpse(siem_data)
    ```

    ::: {.cell-output .cell-output-stdout}

        Rows: 101,904
        Columns: 9
        $ `@timestamp` <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:07.101Z", "…
        $ `@metadata`  <df[,4]> <data.frame[26 x 4]>
        $ event        <df[,4]> <data.frame[26 x 4]>
        $ log          <df[,1]> <data.frame[26 x 1]>
        $ message      <chr> "A token right was adjusted.\n\nSubject:\n\tSecurity I…
        $ winlog       <df[,16]> <data.frame[26 x 16]>
        $ ecs          <df[,1]> <data.frame[26 x 1]>
        $ host         <df[,1]> <data.frame[26 x 1]>
        $ agent        <df[,5]> <data.frame[26 x 5]>

    ::: :::

3.  Раскрытие вложенных списков

    ::: {.cell}

    ``` r
    siem_data_clean <- siem_data %>%
    unnest_wider(winlog, names_sep = "_") %>%
    unnest_wider(event, names_sep = "_") %>%
    unnest_wider(agent, names_sep = "_") %>%
    unnest_wider(host, names_sep = "_") %>%
    unnest_wider(log, names_sep = "_")

    glimpse(siem_data_clean)
    ```

    ::: {.cell-output .cell-output-stdout}

        Rows: 101,904
        Columns: 31
        $ `@timestamp`         <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:07.…
        $ `@metadata`          <df[,4]> <data.frame[26 x 4]>
        $ event_created        <chr> "2019-10-20T20:11:09.988Z", "2019-10-20T20:11:…
        $ event_kind           <chr> "event", "event", "event", "event", "event", "eve…
        $ event_code           <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10, 7…
        $ event_action         <chr> "Token Right Adjusted Events", "Sensitive Privile…
        $ log_level            <chr> "information", "information", "information", "inf…
        $ message              <chr> "A token right was adjusted.\n\nSubject:\n\tSecur…
        $ winlog_event_data    <df[,234]> <data.frame[26 x 234]>
        $ winlog_event_id      <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10, 7…
        $ winlog_provider_name <chr> "Microsoft-Windows-Security-Auditing", "Micr…
        $ winlog_api           <chr> "wineventlog", "wineventlog", "wineventlog", "win…
        $ winlog_record_id     <int> 50588, 104875, 226649, 153525, 163488, 153526, 13…
        $ winlog_computer_name <chr> "HR001.shire.com", "HFDC01.shire.com", "IT001.shi…
        $ winlog_process       <df[,2]> <data.frame[26 x 2]>
        $ winlog_keywords      <list<list>> ["Audit Success"], ["Audit Failure"], [<NULL>], […
        $ winlog_provider_guid <chr> "{54849625-5478-4994-a5ba-3e3b0328c30d}", "{54849…
        $ winlog_channel       <chr> "security", "Security", "Microsoft-Windows-Sys…
        $ winlog_task          <chr> "Token Right Adjusted Events", "Sensitive …
        $ winlog_opcode        <chr> "Info", "Info", "Info", "Info", "Info", "Info", "…
        $ winlog_version       <int> NA, NA, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, NA, 3…
        $ winlog_user          <df[,4]> <data.frame[26 x 4]>
        $ winlog_activity_id   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
        $ winlog_user_data     <df[,30]> <data.frame[26 x 30]>
        $ ecs                  <df[,1]> <data.frame[26 x 1]>
        $ host_name            <chr> "WECServer", "WECServer", "WECServer", "WECServer…
        $ agent_ephemeral_id   <chr> "b372be1f-ba0a-4d7e-b4df-79eac86e1fde", "b372be1f…
        $ agent_hostname       <chr> "WECServer", "WECServer", "WECServer", "WECSer…
        $ agent_id             <chr> "d347d9a4-bff4-476c-b5a4-d51119f78250", "d347d9a4…
        $ agent_version        <chr> "7.4.0", "7.4.0", "7.4.0", "7.4.0", "7.4.0", …
        $ agent_type           <chr> "winlogbeat", "winlogbeat", "winlogbeat", "win…

    ::: :::

4.  Минимизация колонок

    ::: {.cell}

    ``` r
    siem_data_clean <- siem_data_clean %>% select(where(~n_distinct(.) > 1))
    glimpse(siem_data_clean)
    ```

    ::: {.cell-output .cell-output-stdout}

        Rows: 101,904
        Columns: 21
        $ `@timestamp`         <chr> "2019-10-20T20:11:06.937Z", "2019-10-20T20:11:07.…
        $ event_created        <chr> "2019-10-20T20:11:09.988Z", "2019-10-20T20:11:09.…
        $ event_code           <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, 10, 7…
        $ event_action         <chr> "Token Right Adjusted Events", "Sensitive Privile…
        $ log_level            <chr> "information", "information", "information", "inf…
        $ message              <chr> "A token right was adjusted.\n\nSubject:\n\tSecur…
        $ winlog_event_data    <df[,234]> <data.frame[26 x 234]>
        $ winlog_event_id      <int> 4703, 4673, 10, 10, 10, 10, 11, 10, 10, 10, …
        $ winlog_provider_name <chr> "Microsoft-Windows-Security-Auditing", "Microsoft…
        $ winlog_record_id     <int> 50588, 104875, 226649, 153525, 163488, 153526, 13…
        $ winlog_computer_name <chr> "HR001.shire.com", "HFDC01.shire.com", "IT001.shi…
        $ winlog_process       <df[,2]> <data.frame[26 x 2]>
        $ winlog_keywords      <list<list>> ["Audit Success"], ["Audit Failure"], [<NULL>], […
        $ winlog_provider_guid <chr> "{54849625-5478-4994-a5ba-3e3b0328c30d}", "{54…
        $ winlog_channel       <chr> "security", "Security", "Microsoft-Windows…
        $ winlog_task          <chr> "Token Right Adjusted Events", "Sensitive Privile…
        $ winlog_opcode        <chr> "Info", "Info", "Info", "Info", "Info", "Info", "…
        $ winlog_version       <int> NA, NA, 3, 3, 3, 3, 2, 3, 3, 3, 3, 3, 3, 3, NA, 3…
        $ winlog_user          <df[,4]> <data.frame[26 x 4]>
        $ winlog_activity_id   <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
        $ winlog_user_data     <df[,30]> <data.frame[26 x 30]>

    ::: :::

5.  Приведение типов столбцов

    ::: {.cell}

    ``` r
    siem_data_clean <- siem_data_clean %>%
    mutate(
      `@timestamp` = ymd_hms(`@timestamp`),
      event_code = as.integer(event_code),
      winlog_event_id = as.integer(winlog_event_id),
    )
    ```

    :::

6.  Количество хостов

    ::: {.cell}

    ``` r
    host_count <- siem_data_clean %>% summarize(hosts = n_distinct(winlog_computer_name))
    glimpse(host_count)
    ```

    ::: {.cell-output .cell-output-stdout}

        Rows: 1
        Columns: 1
        $ hosts <int> 5

    ::: :::

7.  Подготовка датафрейма с расшифровкой Windows Event_ID

    ::: {.cell}

    ``` r
    windows_event_df <- event_df %>%
    filter(grepl("^[-+]?\\d+$", `Current Windows Event ID`)) %>%
    rename(winlog_event_id = `Current Windows Event ID`, description = `Event Summary`) %>%
    mutate(winlog_event_id = as.integer(winlog_event_id),
    description = as.character(description))
    head(windows_event_df)
    ```

    ::: {.cell-output .cell-output-stdout}

        # A tibble: 6 × 4
          winlog_event_id `Legacy Windows Event ID` `Potential Criticality` description 
                    <int> <chr>                     <chr>                   <chr>       
        1            4618 N/A                       High                    A monitored…
        2            4649 N/A                       High                    A replay at…
        3            4719 612                       High                    System audi…
        4            4765 N/A                       High                    SID History…
        5            4766 N/A                       High                    An attempt …
        6            4794 N/A                       High                    An attempt …

    ::: :::

8.  События с высоким и средним уровнем значимости

``` r
warn_events <- siem_data_clean %>%
filter(log_level %in% c("warning")) #little bad
error_events <- siem_data_clean %>%
filter(log_level %in% c("error")) #very bad
error_count <- nrow(error_events)
warn_count <- nrow(warn_events)

sprintf("1. Количество событий высокого уровня значимости: %d", error_count)
```

    [1] "1. Количество событий высокого уровня значимости: 4"

``` r
sprintf("2. Количество событий среднего уровня значимости: %d", warn_count)
```

    [1] "2. Количество событий среднего уровня значимости: 222"

``` r
sprintf("Всего: %d", error_count+warn_count)          
```

    [1] "Всего: 226"

## Оценка результата

В результате лабораторной работы мы научились анализировать дампы
событий ОС Windows

## Вывод

Таким образом, мы научились работать с определенными функциями и
библиотеками языка R для анализа дампов событий ОС Windows

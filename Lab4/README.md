# Исследование метаданных DNS трафика
Mary

## Цель работы

1.  Закрепить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R
3.  Закрепить навыки исследования метаданных DNS трафика

## Исходные данные

1.  Программное обеспечение Windows 10 Pro
2.  Visual Studio Code с установленными плагинами для работы с языком R
3.  Интерпретатор языка R 4.5.2

## План

1.  Импортировать данные DNS
2.  Добавить пропущенные данные о структуре данных (назначении столбцов)
3.  Преобразовать данные в столбцах в нужный формат
4.  Просмотреть общую структуру данных с помощью функции glimpse()
5.  Проанализировать данные

## Шаги:

1.  Добавьте пропущенные данные о структуре данных (назначении столбцов)

    ``` r
    col_names <- c(
      "ts", "uid", "id.orig_h", "id.orig_p", "id.resp_h", "id.resp_p",
      "proto", "trans_id", "rtt", "query", "qclass", "qclass_name",
      "qtype", "qtype_name", "rcode", "rcode_name", "AA", "TC", "RD", "RA",
      "Z", "answers", "TTLs", "rejected"
    )
    ```

2.  Преобразуйте данные в столбцах в нужный формат

    ``` r
    library(readr)
    library(dplyr)
    ```


        Attaching package: 'dplyr'

        The following objects are masked from 'package:stats':

            filter, lag

        The following objects are masked from 'package:base':

            intersect, setdiff, setequal, union

    ``` r
    library(archive)

    con <- archive_read("https://storage.yandexcloud.net/dataset.ctfsec/dns.zip", file = "dns.log", format = "zip")
    dns <- read_tsv(file = con, col_names = col_names, na = c("-", "(empty)"), comment = "#")
    ```

        Rows: 427935 Columns: 23

        ── Column specification ────────────────────────────────────────────────────────
        Delimiter: "\t"
        chr (10): uid, id.orig_h, id.resp_h, proto, rtt, qclass, qtype, rcode, Z, an...
        dbl  (8): ts, id.orig_p, id.resp_p, trans_id, query, qclass_name, qtype_name...
        lgl  (5): rcode_name, AA, TC, RD, TTLs

        ℹ Use `spec()` to retrieve the full column specification for this data.
        ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    ``` r
    dns <- dns %>%
      mutate(ts = as.POSIXct(ts, origin = "1970-01-01", tz = "UTC"))
    ```

3.  Просмотрите общую структуру данных с помощью функции glimpse()

    ``` r
    library(tibble)

    dns <- dns %>% mutate(ts = as.POSIXct(ts, origin = "1970-01-01", tz = "UTC"))

    glimpse(dns)
    ```

        Rows: 427,935
        Columns: 23
        $ ts          <dttm> 2012-03-16 12:30:05, 2012-03-16 12:30:15, 2012-03-16 12:3…
        $ uid         <chr> "CWGtK431H9XuaTN4fi", "C36a282Jljz7BsbGH", "C36a282Jljz7Bs…
        $ id.orig_h   <chr> "192.168.202.100", "192.168.202.76", "192.168.202.76", "19…
        $ id.orig_p   <dbl> 45658, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 1…
        $ id.resp_h   <chr> "192.168.27.203", "192.168.202.255", "192.168.202.255", "1…
        $ id.resp_p   <dbl> 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137…
        $ proto       <chr> "udp", "udp", "udp", "udp", "udp", "udp", "udp", "udp", "u…
        $ trans_id    <dbl> 33008, 57402, 57402, 57402, 57398, 57398, 57398, 62187, 62…
        $ rtt         <chr> "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\…
        $ query       <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1…
        $ qclass      <chr> "C_INTERNET", "C_INTERNET", "C_INTERNET", "C_INTERNET", "C…
        $ qclass_name <dbl> 33, 32, 32, 32, 32, 32, 32, 32, 32, 32, 33, 33, 33, 12, 12…
        $ qtype       <chr> "SRV", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB…
        $ qtype_name  <dbl> 0, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
        $ rcode       <chr> "NOERROR", NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…
        $ rcode_name  <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
        $ AA          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
        $ TC          <lgl> FALSE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRU…
        $ RD          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
        $ RA          <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1, 1, 0…
        $ Z           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
        $ answers     <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
        $ TTLs        <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…

4.  Сколько участников информационного обмена в сети Доброй Организации?

    ``` r
    is_internal_ip <- function(ip) {
      grepl("^10\\.|^192\\.168\\.|^172\\.(1[6-9]|2[0-9]|3[01])\\.", ip)
    }

    all_ips <- unique(c(dns$`id.orig_h`, dns$`id.resp_h`))

    internal_participants <- all_ips[is_internal_ip(all_ips)]

    num_internal_participants <- length(internal_participants)
    num_internal_participants
    ```

        [1] 1267

5.  Какое соотношение участников обмена внутри сети и участников
    обращений к внешним ресурсам?

    ``` r
    dns <- dns %>%
      mutate(
        orig_internal = is_internal_ip(`id.orig_h`),
        resp_internal = is_internal_ip(`id.resp_h`)
      )

    internal_to_internal <- dns %>% filter(orig_internal & resp_internal)

    internal_to_external <- dns %>% filter(orig_internal & !resp_internal)

    ratio <- nrow(internal_to_internal) / nrow(internal_to_external)
    round(ratio, 3)
    ```

        [1] 16.453

6.  Найдите топ-10 участников сети, проявляющих наибольшую сетевую
    активность

    ``` r
    top_active_users <- dns %>%
      filter(orig_internal) %>%
      count(`id.orig_h`, sort = TRUE) %>%
      head(10)

    top_active_users
    ```

        # A tibble: 10 × 2
           id.orig_h           n
           <chr>           <int>
         1 10.10.117.210   75943
         2 192.168.202.93  26522
         3 192.168.202.103 18121
         4 192.168.202.76  16978
         5 192.168.202.97  16176
         6 192.168.202.141 14967
         7 10.10.117.209   14222
         8 192.168.202.110 13372
         9 192.168.203.63  12148
        10 192.168.202.106 10784

7.  Найдите топ-10 доменов, к которым обращаются пользователи сети и
    соответственное количество обращений (?)

    ``` r
    top_domains <- dns %>%
      filter(!is.na(query)) %>%
      count(query, sort = TRUE) %>%
      head(10)

    top_domains
    ```

        # A tibble: 3 × 2
          query      n
          <dbl>  <int>
        1     1 422339
        2     3   1016
        3 32769    932

8.  Опеределите базовые статистические характеристики (функция
    summary()) интервала времени между последовательными обращениями к
    топ-10 доменам

    ``` r
    library(lubridate)
    ```


        Attaching package: 'lubridate'

        The following objects are masked from 'package:base':

            date, intersect, setdiff, union

    ``` r
    top10_list <- top_domains$query

    intervals_data <- dns %>%
      filter(query %in% top10_list) %>%
      arrange(query, ts) %>%
      group_by(query) %>%
      mutate(delta_sec = as.numeric(difftime(ts, lag(ts), units = "secs"))) %>%
      ungroup() %>%
      filter(!is.na(delta_sec))

    summary(intervals_data$delta_sec)
    ```

             Min.   1st Qu.    Median      Mean   3rd Qu.      Max. 
            0.000     0.000     0.010     0.791     0.150 58362.950 

9.  Поиск признаков скрытого DNS-канала (периодические запросы)

    ``` r
    periodic_check <- dns %>%
      filter(orig_internal, !is.na(query)) %>%
      select(`id.orig_h`, query, ts) %>%
      arrange(`id.orig_h`, query, ts) %>%
      group_by(`id.orig_h`, query) %>%
      mutate(
        delta = as.numeric(difftime(ts, lag(ts), units = "secs")),
        n = n()
      ) %>%
      filter(n >= 5) %>%
      summarise(
        mean_delta = mean(delta, na.rm = TRUE),
        sd_delta = sd(delta, na.rm = TRUE),
        .groups = "drop"
      ) %>%
      mutate(cv = sd_delta / mean_delta) %>%
      filter(cv < 0.1 & mean_delta > 10)

    if (nrow(periodic_check) > 0) {
      cat("Обнаружены подозрительные IP с периодическими запросами:\n")
      print(unique(periodic_check$`id.orig_h`))
    } else {
      cat("Подозрительных периодических DNS-запросов не обнаружено.\n")
    }
    ```

        Подозрительных периодических DNS-запросов не обнаружено.

## Оценка результата

В результате лабораторной работы мы развили практические навыки
использования языка программирования R для обработки данных и закрепили
знания базовых типов данных языка R

## Вывод

Таким образом, мы научились, используя программный пакет dplyr,
анализировать DNS-логи с помощью языка программирования R

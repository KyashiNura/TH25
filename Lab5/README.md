# Исследование информации о состоянии беспроводных сетей
Mary

## Цель работы

1.  Получить знания о методах исследования радиоэлектронной обстановки.
2.  Составить представление о механизмах работы Wi-Fi сетей на канальном
    и сетевом уровне модели OSI.
3.  Закрепить практические навыки использования языка программирования R
    для обработки данных
4.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R

## Исходные данные

1.  Программное обеспечение Windows 10 Pro
2.  Visual Studio Code с установленными плагинами для работы с языком R
3.  Интерпретатор языка R 4.5.2

## План:

1.  Загрузить данные и провести действи для приведения в “аккуратный”
    вид
2.  Определить небезопасные точки доступа
3.  Выявить устройства, использующие последнюю версию протокола
    шифрования WPA3, и названия точек доступа, реализованных на этих
    устройствах
4.  Отсортировать точки доступа по интервалу времени, в течение которого
    они находились на связи, по убыванию.
5.  Обнаружить топ-10 самых быстрых точек доступа.
6.  Отсортировать точки доступа по частоте отправки запросов (beacons) в
    единицу времени по их убыванию.
7.  Определить производителя для каждого обнаруженного устройства
8.  Обнаружить устройства, которые НЕ рандомизируют свой MAC адрес
9.  Кластеризовать запросы от устройств к точкам доступа по их именам.
    Определить время появления устройства в зоне радиовидимости и время
    выхода его из нее.
10. Оценить стабильность уровня сигнала внури кластера во времени.
    Выявить наиболее стабильный кластер.

## Шаги:

1.  Импорт библиотек

``` r
library(tidyverse)
```

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ dplyr     1.1.4     ✔ readr     2.1.6
    ✔ forcats   1.0.1     ✔ stringr   1.6.0
    ✔ ggplot2   4.0.1     ✔ tibble    3.3.0
    ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ✔ purrr     1.2.0     
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(lubridate)
```

1.  Чтение и разделение данных

    ``` r
    lines <- read_lines("P2_wifi_data.csv")
    split_idx <- which(lines == "")[1]

    ap_lines <- lines[3:(split_idx-1)]
    ap_data <- read_csv(paste(ap_lines, collapse = "\n"),
      col_names = c("BSSID", "First_time_seen", "Last_time_seen", "channel", "Speed", 
                    "Privacy", "Cipher", "Authentication", "Power", "#beacons", "#IV", 
                    "LAN_IP", "ID_length", "ESSID", "Key"),
      na = c("", " ", "(not associated)")
    )
    ```

        Rows: 2 Columns: 15
        ── Column specification ────────────────────────────────────────────────────────
        Delimiter: ","
        chr (15): BSSID, First_time_seen, Last_time_seen, channel, Speed, Privacy, C...

        ℹ Use `spec()` to retrieve the full column specification for this data.
        ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

    ``` r
    sta_lines <- lines[(split_idx+2):length(lines)]
    sta_data <- read_csv(paste(sta_lines, collapse = "\n"),
      col_names = c("Station_MAC", "First_time_seen", "Last_time_seen", 
                    "Power", "Packets", "BSSID", "Probed_ESSIDs"),
      na = c("", " ", "(not associated)")
    )
    ```

        Warning: One or more parsing issues, call `problems()` on your data frame for details,
        e.g.:
          dat <- vroom(...)
          problems(dat)

        Rows: 12249 Columns: 15
        ── Column specification ────────────────────────────────────────────────────────
        Delimiter: ","
        chr  (10): Station_MAC, BSSID, Probed_ESSIDs, X8, X9, X10, X11, X12, X13, X14
        dbl   (2): Power, Packets
        lgl   (1): X15
        dttm  (2): First_time_seen, Last_time_seen

        ℹ Use `spec()` to retrieve the full column specification for this data.
        ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

2.  Привести датасеты в вид “аккуратных данных”, преобразовать типы
    столбцов в соответствии с типом данных

    ``` r
    ap_data
    ```

        # A tibble: 2 × 15
          BSSID             First_time_seen  Last_time_seen channel Speed Privacy Cipher
          <chr>             <chr>            <chr>          <chr>   <chr> <chr>   <chr> 
        1 BE:F1:71:D5:17:8B 2023-07-28 09:1… 2023-07-28 11… 1       195   WPA2    CCMP  
        2 BSSID             First time seen  Last time seen channel Speed Privacy Cipher
        # ℹ 8 more variables: Authentication <chr>, Power <chr>, `#beacons` <chr>,
        #   `#IV` <chr>, LAN_IP <chr>, ID_length <chr>, ESSID <chr>, Key <chr>

    ``` r
    sta_data
    ```

        # A tibble: 12,249 × 15
           Station_MAC       First_time_seen     Last_time_seen      Power Packets BSSID
           <chr>             <dttm>              <dttm>              <dbl>   <dbl> <chr>
         1 BE:F1:71:D5:17:8B 2023-07-28 09:13:03 2023-07-28 11:50:50     1     195 WPA2 
         2 6E:C7:EC:16:DA:1A 2023-07-28 09:13:03 2023-07-28 11:55:12     1     130 WPA2 
         3 9A:75:A8:B9:04:1E 2023-07-28 09:13:03 2023-07-28 11:53:31     1     360 WPA2 
         4 4A:EC:1E:DB:BF:95 2023-07-28 09:13:03 2023-07-28 11:04:01     7     360 WPA2 
         5 D2:6D:52:61:51:5D 2023-07-28 09:13:03 2023-07-28 10:30:19     6     130 WPA2 
         6 E8:28:C1:DC:B2:52 2023-07-28 09:13:03 2023-07-28 11:55:38     6     130 OPN  
         7 BE:F1:71:D6:10:D7 2023-07-28 09:13:03 2023-07-28 11:50:44    11     195 WPA2 
         8 0A:C5:E1:DB:17:7B 2023-07-28 09:13:03 2023-07-28 11:36:31    11     130 WPA2 
         9 38:1A:52:0D:84:D7 2023-07-28 09:13:03 2023-07-28 10:25:02    11     130 WPA2 
        10 BE:F1:71:D5:0E:53 2023-07-28 09:13:03 2023-07-28 10:29:21     1     195 WPA2 
        # ℹ 12,239 more rows
        # ℹ 9 more variables: Probed_ESSIDs <chr>, X8 <chr>, X9 <chr>, X10 <chr>,
        #   X11 <chr>, X12 <chr>, X13 <chr>, X14 <chr>, X15 <lgl>

    ``` r
    ap_data_clean <- ap_data %>% mutate(
      BSSID = as.character(BSSID),
      First_time_seen = ymd_hms(First_time_seen),
      Last_time_seen = ymd_hms(Last_time_seen),
      channel = as.integer(channel),
      Speed = as.integer(Speed),
      Privacy = as.character(Privacy),
      Cipher = as.character(Cipher),
      Authentication = as.character(Authentication),
      Power = as.integer(Power),
      Beacons = as.integer(`#beacons`),
      IV = as.integer(`#IV`),
      LAN_IP = as.character(LAN_IP),
      ID_length = as.integer(ID_length),
      ESSID = as.character(ESSID)
    )
    ```

        Warning: There were 8 warnings in `mutate()`.
        The first warning was:
        ℹ In argument: `First_time_seen = ymd_hms(First_time_seen)`.
        Caused by warning:
        !  1 failed to parse.
        ℹ Run `dplyr::last_dplyr_warnings()` to see the 7 remaining warnings.

    ``` r
    sta_data_clean <- sta_data %>% mutate(
      Station_MAC = as.character(Station_MAC),
      First_time_seen = ymd_hms(First_time_seen),
      Last_time_seen = ymd_hms(Last_time_seen),
      Power = as.integer(Power),
      Packets = as.integer(Packets),
      BSSID = as.character(BSSID),
      Probed_ESSIDs = as.character(Probed_ESSIDs)
    )
    ```

3.  Определить небезопасные точки доступа (без шифрования – OPN)

    ``` r
    insecure_aps <- ap_data_clean %>% filter(Privacy == "OPN" | is.na(Privacy))
    insecure_aps 
    ```

        # A tibble: 0 × 17
        # ℹ 17 variables: BSSID <chr>, First_time_seen <dttm>, Last_time_seen <dttm>,
        #   channel <int>, Speed <int>, Privacy <chr>, Cipher <chr>,
        #   Authentication <chr>, Power <int>, #beacons <chr>, #IV <chr>, LAN_IP <chr>,
        #   ID_length <int>, ESSID <chr>, Key <chr>, Beacons <int>, IV <int>

4.  Определить производителя для каждого обнаруженного устройства

    ``` r
    extract_oui <- function(mac_vec) {
      cleaned_mac <- gsub("[:.-]", "", toupper(mac_vec))
      valid_macs <- str_sub(cleaned_mac, start = 1, end = 6)[nchar(cleaned_mac) >= 6]  
      invalid_macs <- rep(NA_character_, sum(nchar(cleaned_mac) < 6))  
      c(valid_macs, invalid_macs)
    }

    oui_url <- "https://gitlab.com/wireshark/wireshark/-/raw/release-4.0/manuf"
    oui_raw <- readLines(oui_url)
    oui_clean <- oui_raw[str_detect(oui_raw, regex("^[0-9A-F]{2}:[0-9A-F]{2}:[0-9A-F]{2}", ignore_case = TRUE))]
    oui_df <- tibble(line = oui_clean) %>%
      separate(line, into = c("OUI", "ShortName", "FullName"), sep = "\\s+", fill = "right", extra = "merge") %>%
      mutate(OUI = toupper(gsub(":", "", OUI))) %>% filter(str_length(OUI) == 6)

    ap_data_clean <- ap_data_clean %>% mutate(OUI = extract_oui(BSSID))
    sta_data_clean <- sta_data_clean %>% mutate(OUI = extract_oui(Station_MAC))

    ap_with_vendor <- ap_data_clean %>% left_join(oui_df, by = "OUI")
    sta_with_vendor <- sta_data_clean %>% left_join(oui_df, by = "OUI")
    View(ap_with_vendor)
    View(sta_with_vendor)
    ```

5.  Выявить устройства, использующие последнюю версию протокола
    шифрования WPA3, и названия точек доступа, реализованных на этих
    устройствах

    ``` r
    wpa3_aps <- ap_data_clean %>% filter(str_detect(Privacy, regex("WPA3", ignore_case = TRUE)))
    wpa3_aps
    ```

        # A tibble: 0 × 18
        # ℹ 18 variables: BSSID <chr>, First_time_seen <dttm>, Last_time_seen <dttm>,
        #   channel <int>, Speed <int>, Privacy <chr>, Cipher <chr>,
        #   Authentication <chr>, Power <int>, #beacons <chr>, #IV <chr>, LAN_IP <chr>,
        #   ID_length <int>, ESSID <chr>, Key <chr>, Beacons <int>, IV <int>, OUI <chr>

6.  Отсортировать точки доступа по интервалу времени, в течение которого
    они находились на связи, по убыванию.

    ``` r
    ap_data_clean <- ap_data_clean %>% mutate(Duration_sec = as.numeric(difftime(Last_time_seen, First_time_seen, units = "secs"))) %>% arrange(desc(Duration_sec))
    ap_data_clean
    ```

        # A tibble: 2 × 19
          BSSID     First_time_seen     Last_time_seen      channel Speed Privacy Cipher
          <chr>     <dttm>              <dttm>                <int> <int> <chr>   <chr> 
        1 BE:F1:71… 2023-07-28 09:13:03 2023-07-28 11:50:50       1   195 WPA2    CCMP  
        2 BSSID     NA                  NA                       NA    NA Privacy Cipher
        # ℹ 12 more variables: Authentication <chr>, Power <int>, `#beacons` <chr>,
        #   `#IV` <chr>, LAN_IP <chr>, ID_length <int>, ESSID <chr>, Key <chr>,
        #   Beacons <int>, IV <int>, OUI <chr>, Duration_sec <dbl>

7.  Обнаружить топ-10 самых быстрых точек доступа.

    ``` r
    top_speed_aps <- ap_data_clean %>% arrange(desc(Speed)) %>% head(10)
    top_speed_aps
    ```

        # A tibble: 2 × 19
          BSSID     First_time_seen     Last_time_seen      channel Speed Privacy Cipher
          <chr>     <dttm>              <dttm>                <int> <int> <chr>   <chr> 
        1 BE:F1:71… 2023-07-28 09:13:03 2023-07-28 11:50:50       1   195 WPA2    CCMP  
        2 BSSID     NA                  NA                       NA    NA Privacy Cipher
        # ℹ 12 more variables: Authentication <chr>, Power <int>, `#beacons` <chr>,
        #   `#IV` <chr>, LAN_IP <chr>, ID_length <int>, ESSID <chr>, Key <chr>,
        #   Beacons <int>, IV <int>, OUI <chr>, Duration_sec <dbl>

8.  Отсортировать точки доступа по частоте отправки запросов (beacons) в
    единицу времени по их убыванию.

    ``` r
    beacon_rate <- ap_data_clean %>% mutate(Duration_min = Duration_sec / 60, Beacons_per_min = Beacons / pmax(Duration_min, 1)) %>% arrange(desc(Beacons_per_min))
    beacon_rate
    ```

        # A tibble: 2 × 21
          BSSID     First_time_seen     Last_time_seen      channel Speed Privacy Cipher
          <chr>     <dttm>              <dttm>                <int> <int> <chr>   <chr> 
        1 BE:F1:71… 2023-07-28 09:13:03 2023-07-28 11:50:50       1   195 WPA2    CCMP  
        2 BSSID     NA                  NA                       NA    NA Privacy Cipher
        # ℹ 14 more variables: Authentication <chr>, Power <int>, `#beacons` <chr>,
        #   `#IV` <chr>, LAN_IP <chr>, ID_length <int>, ESSID <chr>, Key <chr>,
        #   Beacons <int>, IV <int>, OUI <chr>, Duration_sec <dbl>, Duration_min <dbl>,
        #   Beacons_per_min <dbl>

9.  Определить производителя для каждого обнаруженного устройства

    ``` r
    sta_with_vendor
    ```

        # A tibble: 12,249 × 18
           Station_MAC       First_time_seen     Last_time_seen      Power Packets BSSID
           <chr>             <dttm>              <dttm>              <int>   <int> <chr>
         1 BE:F1:71:D5:17:8B 2023-07-28 09:13:03 2023-07-28 11:50:50     1     195 WPA2 
         2 6E:C7:EC:16:DA:1A 2023-07-28 09:13:03 2023-07-28 11:55:12     1     130 WPA2 
         3 9A:75:A8:B9:04:1E 2023-07-28 09:13:03 2023-07-28 11:53:31     1     360 WPA2 
         4 4A:EC:1E:DB:BF:95 2023-07-28 09:13:03 2023-07-28 11:04:01     7     360 WPA2 
         5 D2:6D:52:61:51:5D 2023-07-28 09:13:03 2023-07-28 10:30:19     6     130 WPA2 
         6 E8:28:C1:DC:B2:52 2023-07-28 09:13:03 2023-07-28 11:55:38     6     130 OPN  
         7 BE:F1:71:D6:10:D7 2023-07-28 09:13:03 2023-07-28 11:50:44    11     195 WPA2 
         8 0A:C5:E1:DB:17:7B 2023-07-28 09:13:03 2023-07-28 11:36:31    11     130 WPA2 
         9 38:1A:52:0D:84:D7 2023-07-28 09:13:03 2023-07-28 10:25:02    11     130 WPA2 
        10 BE:F1:71:D5:0E:53 2023-07-28 09:13:03 2023-07-28 10:29:21     1     195 WPA2 
        # ℹ 12,239 more rows
        # ℹ 12 more variables: Probed_ESSIDs <chr>, X8 <chr>, X9 <chr>, X10 <chr>,
        #   X11 <chr>, X12 <chr>, X13 <chr>, X14 <chr>, X15 <lgl>, OUI <chr>,
        #   ShortName <chr>, FullName <chr>

10. Обнаружить устройства, которые НЕ рандомизируют свой MAC адрес

    ``` r
    is_global_mac <- function(mac_vec) {
      mac_clean <- gsub("[:.-]", "", toupper(mac_vec))
      first_byte_hex <- substr(mac_clean, 1, 2)
      first_byte_dec <- suppressWarnings(strtoi(first_byte_hex, base = 16))
      is_global <- (first_byte_dec & 2) == 0
      is_global[is.na(is_global)] <- FALSE
      return(is_global)
    }

    sta_data_clean <- sta_data_clean %>% mutate(is_global = is_global_mac(Station_MAC))

    non_randomized_clients <- sta_data_clean %>% group_by(Station_MAC) %>% summarise(
      probed_networks = n_distinct(Probed_ESSIDs[Probed_ESSIDs != "Not Probing"], na.rm = TRUE),
      observation_count = n(),
      first_seen = min(First_time_seen),
      last_seen = max(Last_time_seen),
      total_packets = sum(Packets, na.rm = TRUE),
      .groups = "drop"
    ) %>% filter(probed_networks > 1 | observation_count > 5) %>% arrange(desc(probed_networks), desc(observation_count))

    non_randomized_clients
    ```

        # A tibble: 0 × 6
        # ℹ 6 variables: Station_MAC <chr>, probed_networks <int>,
        #   observation_count <int>, first_seen <dttm>, last_seen <dttm>,
        #   total_packets <int>

11. Кластеризовать запросы от устройств к точкам доступа по их именам.
    Определить время появления устройства в зоне радиовидимости и время
    выхода его из нее.

    ``` r
    sta_clusters <- sta_data_clean %>% filter(BSSID != "(not associated)" & !is.na(Probed_ESSIDs) & Probed_ESSIDs != "") %>% group_by(BSSID, Probed_ESSIDs) %>% summarise(
      mean_power = mean(Power, na.rm = TRUE),
      sd_power = sd(Power, na.rm = TRUE),
      n = n(),
      first_seen = min(First_time_seen),
      last_seen = max(Last_time_seen),
      .groups = "drop"
    ) %>% arrange(sd_power)

    sta_clusters
    ```

        # A tibble: 52 × 7
           BSSID             Probed_ESSIDs mean_power sd_power     n first_seen         
           <chr>             <chr>              <dbl>    <dbl> <int> <dttm>             
         1 1E:93:E3:1B:3C:F4 Galaxy A71        -48.5     0.707     2 2023-07-28 09:13:13
         2 E8:28:C1:DC:B2:50 MIREA_HOTSPOT     -63       2.83      2 2023-07-28 09:45:48
         3 E8:28:C1:DD:04:40 MIREA_HOTSPOT     -61       2.83      2 2023-07-28 10:14:39
         4 8E:55:4A:85:5B:01 Vladimir          -51.5     4.12      4 2023-07-28 09:31:57
         5 00:26:99:F2:7A:E2 GIVC              -64.7     4.54      7 2023-07-28 09:13:06
         6 00:26:99:BA:75:80 GIVC              -60.3     5.32      6 2023-07-28 09:39:02
         7 E8:28:C1:DC:B2:50 MIREA_GUESTS      -57.7     5.77      3 2023-07-28 10:35:02
         8 WPA2              CCMP                6.67    6.41     72 2023-07-28 09:13:03
         9 AA:F4:3F:EE:49:0B Redmi Note 8…     -49       8.49      2 2023-07-28 09:19:44
        10 E8:28:C1:DC:F0:90 MIREA_GUESTS      -63       8.49      2 2023-07-28 10:30:39
        # ℹ 42 more rows
        # ℹ 1 more variable: last_seen <dttm>

12. Оценить стабильность уровня сигнала внури кластера во времени.
    Выявить наиболее стабильный кластер.

    ``` r
    sta_clusters <- sta_clusters %>% mutate(signal_stability = 1 / (sd_power + 1e-6))
    most_stable_cluster <- sta_clusters %>% arrange(desc(signal_stability)) %>% slice(1)
    most_stable_cluster
    ```

        # A tibble: 1 × 8
          BSSID             Probed_ESSIDs mean_power sd_power     n first_seen         
          <chr>             <chr>              <dbl>    <dbl> <int> <dttm>             
        1 1E:93:E3:1B:3C:F4 Galaxy A71         -48.5    0.707     2 2023-07-28 09:13:13
        # ℹ 2 more variables: last_seen <dttm>, signal_stability <dbl>

## Оценка результата

В результате лабораторной работы мы получили и закрепили знания о
методах исследования радиоэлектронной обстановки

## Вывод

Таким образом, мы научились, используя программный пакет tidyverse,
анализировать сетевые дампы с помощью языка программирования R

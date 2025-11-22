# Основы обработки данных с помощью R и Dplyr
Mary

## Цель работы

1.  Развить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания базовых типов данных языка R
3.  Развить практические навыки использования функций обработки данных
    пакета dplyr – функции select(), filter(), mutate(), arrange(),
    group_by()

## Исходные данные

1.  Программное обеспечение Windows 10 Pro
2.  Visual Studio Code с установленными плагинами для работы с языком R
3.  Интерпретатор языка R 4.5.2

## План

Проанализировать встроенные в пакет nycflights13 наборы данных с помощью
языка R и ответить на вопросы

## Шаги:

1.  Загрузка библиотек

    ``` r
    library(nycflights13)
    library(dplyr)
    ```


        Attaching package: 'dplyr'

        The following objects are masked from 'package:stats':

            filter, lag

        The following objects are masked from 'package:base':

            intersect, setdiff, setequal, union

2.  Сколько встроенных в пакет nycflights13 датафреймов? 5

    ``` r
    length(ls("package:nycflights13"))
    ```

        [1] 5

3.  Сколько строк в каждом датафрейме? 336776 16 1458 3322 26115

    ``` r
    sapply(list(flights, airlines, airports, planes, weather), nrow)
    ```

        [1] 336776     16   1458   3322  26115

4.  Сколько столбцов в каждом датафрейме? 19 2 8 9 15

    ``` r
    sapply(list(flights, airlines, airports, planes, weather), ncol)
    ```

        [1] 19  2  8  9 15

5.  Как просмотреть примерный вид датафрейма?

    ``` r
    glimpse(flights)
    ```

        Rows: 336,776
        Columns: 19
        $ year           <int> 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2…
        $ month          <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1…
        $ day            <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1…
        $ dep_time       <int> 517, 533, 542, 544, 554, 554, 555, 557, 557, 558, 558, …
        $ sched_dep_time <int> 515, 529, 540, 545, 600, 558, 600, 600, 600, 600, 600, …
        $ dep_delay      <dbl> 2, 4, 2, -1, -6, -4, -5, -3, -3, -2, -2, -2, -2, -2, -1…
        $ arr_time       <int> 830, 850, 923, 1004, 812, 740, 913, 709, 838, 753, 849,…
        $ sched_arr_time <int> 819, 830, 850, 1022, 837, 728, 854, 723, 846, 745, 851,…
        $ arr_delay      <dbl> 11, 20, 33, -18, -25, 12, 19, -14, -8, 8, -2, -3, 7, -1…
        $ carrier        <chr> "UA", "UA", "AA", "B6", "DL", "UA", "B6", "EV", "B6", "…
        $ flight         <int> 1545, 1714, 1141, 725, 461, 1696, 507, 5708, 79, 301, 4…
        $ tailnum        <chr> "N14228", "N24211", "N619AA", "N804JB", "N668DN", "N394…
        $ origin         <chr> "EWR", "LGA", "JFK", "JFK", "LGA", "EWR", "EWR", "LGA",…
        $ dest           <chr> "IAH", "IAH", "MIA", "BQN", "ATL", "ORD", "FLL", "IAD",…
        $ air_time       <dbl> 227, 227, 160, 183, 116, 150, 158, 53, 140, 138, 149, 1…
        $ distance       <dbl> 1400, 1416, 1089, 1576, 762, 719, 1065, 229, 944, 733, …
        $ hour           <dbl> 5, 5, 5, 5, 6, 5, 6, 6, 6, 6, 6, 6, 6, 6, 6, 5, 6, 6, 6…
        $ minute         <dbl> 15, 29, 40, 45, 0, 58, 0, 0, 0, 0, 0, 0, 0, 0, 0, 59, 0…
        $ time_hour      <dttm> 2013-01-01 05:00:00, 2013-01-01 05:00:00, 2013-01-01 0…

    ``` r
    head(flights, 10)
    ```

        # A tibble: 10 × 19
            year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time
           <int> <int> <int>    <int>          <int>     <dbl>    <int>          <int>
         1  2013     1     1      517            515         2      830            819
         2  2013     1     1      533            529         4      850            830
         3  2013     1     1      542            540         2      923            850
         4  2013     1     1      544            545        -1     1004           1022
         5  2013     1     1      554            600        -6      812            837
         6  2013     1     1      554            558        -4      740            728
         7  2013     1     1      555            600        -5      913            854
         8  2013     1     1      557            600        -3      709            723
         9  2013     1     1      557            600        -3      838            846
        10  2013     1     1      558            600        -2      753            745
        # ℹ 11 more variables: arr_delay <dbl>, carrier <chr>, flight <int>,
        #   tailnum <chr>, origin <chr>, dest <chr>, air_time <dbl>, distance <dbl>,
        #   hour <dbl>, minute <dbl>, time_hour <dttm>

6.  Сколько компаний-перевозчиков (carrier) учитывают эти наборы данных
    (представлено в наборах данных)? 16

    ``` r
    flights %>%
      distinct(carrier) %>%
      nrow()
    ```

        [1] 16

7.  Сколько рейсов принял аэропорт John F Kennedy Intl в мае? 0

    ``` r
    flights %>%
      filter(dest == "JFK", month == 5) %>%
      nrow()
    ```

        [1] 0

8.  Какой самый северный аэропорт? Dillant Hopkins Airport

    ``` r
    airports %>%
      arrange(desc(lat)) %>%
      slice(1) %>%
      pull(name)
    ```

        [1] "Dillant Hopkins Airport"

9.  Какой аэропорт самый высокогорный (находится выше всех над уровнем
    моря)? Telluride

    ``` r
    airports %>%
      arrange(desc(alt)) %>%
      slice(1) %>%
      pull(name)
    ```

        [1] "Telluride"

10. Какие бортовые номера у самых старых самолетов? “N381AA” “N201AA”
    “N567AA” “N378AA” “N575AA”

    ``` r
    planes %>%
      filter(!is.na(year)) %>%
      arrange(year) %>%
      slice(1:5) %>%
      pull(tailnum)
    ```

        [1] "N381AA" "N201AA" "N567AA" "N378AA" "N575AA"

11. Какая средняя температура воздуха была в сентябре в аэропорту John F
    Kennedy Intl (в градусах Цельсия)? 19.38764

    ``` r
    weather %>%
      filter(origin == "JFK", month == 9) %>%
      summarise(avg_temp_C = mean((temp - 32) * 5/9, na.rm = TRUE)) %>%
      pull(avg_temp_C)
    ```

        [1] 19.38764

12. Самолеты какой авиакомпании совершили больше всего вылетов в июне?
    UA

    ``` r
    flights %>%
      filter(month == 6) %>%
      count(carrier, sort = TRUE) %>%
      slice(1) %>%
      pull(carrier)
    ```

        [1] "UA"

13. Самолеты какой авиакомпании задерживались чаще других в 2013 году?
    WN

    ``` r
    flights %>%
      filter(!is.na(dep_delay)) %>%
      group_by(carrier) %>%
      summarise(delay_rate = mean(dep_delay > 0, na.rm = TRUE)) %>%
      arrange(desc(delay_rate)) %>%
      slice(1) %>%
      pull(carrier)
    ```

        [1] "WN"

## Оценка результата

В результате лабораторной работы мы развили практические навыки
использования языка программирования R для обработки данных и закрепили
знания базовых типов данных языка R

## Вывод

Таким образом, мы научились использовать функции обработки данных пакета
dplyr – функции select(), filter(), mutate(), arrange(), group_by()

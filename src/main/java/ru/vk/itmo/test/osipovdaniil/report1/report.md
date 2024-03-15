# Отчёт по 1 заданию.

Скрипты для теста методов лежат в папке `scripts`. Последовательные `put, get, delete`. <br>
`flushThresholdBytes = 2^20`<br>
Реализация `Dao` взята с референса.

## wrk2

Собственно сразу пошёл искать точку разладки запуская 
``./wrk -d 30 -t 1 -c 1 -R 18500 -L -s /home/sbread/Desktop/labs/HLS/2024-highload-dht/src/main/java/ru/vk/itmo/test/osipovdaniil/scripts/put.lua http://localhost:8080``
с разным количеством запросов в секунду. <br>
Так я пришёл к константе `18500` запросов в секунду. `mean latency` было меньше секунды.
На тот момент было 80 SSTabl'ов. (выводы wrk лежат в 
соответсвующей папке в файле с соответствующем названием) <br>

Однако решив перетестировать, заметил увеличение таймингов. <br>
Так обнаружил несколько зависимостей:
1) Скорость работы как `put` так и `get` падает с увеличением количевства SSTable'ов.
2) `get` `delete` `get`. В такой последовательности второй `get` работает быстрее первого. <br>
Логично ведь обновлённые данные лежат выше.

Это было подтверждено, после сброса данных, вызовом скриптов в соотетсвующих последовательностях. <br>
Также было замечен стабильный экспоненциальный скачёк задержек на 75, 90 и выше персентилях. 
Как будто происходит накопление нагрузки.

## async-profiler

Результаты в папке `asyncprof` <br>

С точки зрения cpu основные ресурсы съедает обработка запроса.
Однако есть концептуальная разница в распределнии, так `get` съедает больше ресурсов за счёт возвращения результата.

С точки зрения аллокаций, распределение схожее, за исключением детали, что
в случае `put` и `delete` память аллоцируется на запись, 
а в случае `get` на возвращения результата 
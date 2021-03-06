### WinAPI и DLL
# Лабораторная работа

## Теория

Необходимо разработать приложение, которое умеет строить бары по “приходящим тикам с биржи”.

Примеры графиков из CQG IC показаны ниже. Здесь каждый столбик – это бар.
Горизонтальные чёрточки слева/справа означают цену открытия/закрытия бара.
Верхнее/нижнее значения бара означают максимум/минимум цены внутри бара.

Тики могут группироваться по времени (минуты, часы, дни, месяцы и тд.).
На графике показаны дневные бары. 

![Daily Bar](images/chart_time.png)

Другой способ – это группировка по количеству тиков. Например, тут  каждый бар содержит 100 тиков:

![Constant Volume Bar](images/chart_volume.png)

## Задание

Ваше приложение, для простоты, должно быть консольным.
Тики должны группироваться в бары по времени.
Промежуток для группирования баров (time internval) в секундах должен задаваться пользователем при запуске приложения.
Начало отсчета time interval - время первого тика.
Бары должны печататься в текстовом виде в консоли
(OHLC (open, high, low, close) – цена открытия, максимум, минимум и цена закрытия).

Формат логов:
* Для нового тика: цена @ время
* Для нового бара: Bar closed [Open; High; Low; Close]

Пример (Time Interval = 20 секунд):
```
>> 1253 @ 16:24:37
>> 1235 @ 16:24:42
>> 1257 @ 16:24:49
>> Bar closed [1253; 1257; 1235; 1257]
>> 1258 @ 16:24:58
>> 1256 @ 16:25:05
>> 1254 @ 16:25:12
>> Bar closed [1258; 1258; 1254; 1254]
>> 1280 @ 16:25:35
```

Тики посылает специальная функция, содержащаяся в dll-файле.

Вам необходимо создать DLL с помощью .cpp и .h, приложенных к заданию.
Не нужно менять эти файлы. Нужно создать проект (добавив туда эти файлы),
который скомпилируется в dll.
Смотри папку attachments.

После чего создать **отдельный** exe-файл, который работает с этой DLL.
Для получения данных необходимо:
1. Связать неявно dll-файл с приложением (подробнее см. ниже).
2. Вызвать функцию `StartFeed()`, которая запускает генератор тиков.
3. Считать тики в структуру `Tick` при помощи функции GetMessage.
``` cpp
#include "TickerPlant.h" // contains struct Tick
#include "windows.h"

MSG msg; 
GetMessage(&msg, NULL, 0, 0);
```
В поле `lParam` структуры `msg` будет храниться указатель на структуру `Tick`.
Необходимо сделать `reinterpret_cast<Tick*>(msg.lParam)`, чтобы получить этот указатель.

4. Генератор тиков выделяет память под отправляемую структуру, поэтому вам необходимо её освобождать.
5. По окончании приёма тиков, вызвать функцию `StopFeed()`.

## Дополнительные сведения:

Для перевода времени в строковый формат можно использовать функцию
``` cpp
size_t strftime(char* ptr, size_t maxsize, const char* format, const struct tm* timeptr);
```

В качестве `format` можно указать строку `"%X"` для вывода времени в виде ЧЧ:ММ:СС.

Для представления времени в секундах можно использовать функцию
``` cpp
time_t mktime(struct tm * timeptr);
```

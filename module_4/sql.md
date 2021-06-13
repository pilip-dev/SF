## Закономерности данных
### Задание 4.1
```sql
SELECT ap.city,
       count(ap.airport_code) AS airport_count
FROM dst_project.airports ap
GROUP BY 1
HAVING count(ap.airport_code) > 1
ORDER BY 2 DESC;
```
### Задание 4.2
**Вопрос 1.** 
Таблица рейсов содержит всю информацию о прошлых, текущих и запланированных рейсах. Сколько всего статусов для рейсов определено в таблице?
```sql
SELECT count(DISTINCT f.status)
FROM dst_project.flights AS f
```
**Вопрос 2.** 
Какое количество самолетов находятся в воздухе на момент среза в базе (статус рейса «самолёт уже вылетел и находится в воздухе»).
```sql
SELECT count(*) AS departed
FROM dst_project.flights AS f
WHERE f.status = 'Departed'
```
**Вопрос 3.** 
Места определяют схему салона каждой модели. Сколько мест имеет самолет модели  (Boeing 777-300)?
```sql
SELECT count(DISTINCT s.seat_no) AS seat_count
FROM dst_project.aircrafts ac
JOIN dst_project.seats s ON s.aircraft_code=ac.aircraft_code
WHERE ac.model='Boeing 777-300'
```
**Вопрос 4.**
Сколько состоявшихся (фактических) рейсов было совершено между 1 апреля 2017 года и 1 сентября 2017 года?
```sql
SELECT count(f.flight_id) AS arrived
FROM dst_project.flights AS f
WHERE f.status='Arrived'
  AND f.actual_arrival BETWEEN '2017-04-01'::date AND '2017-09-01'::date
```

### Задание 4.3
**Вопрос 1.**
Сколько всего рейсов было отменено по данным базы?
```sql
SELECT count(f.flight_id) AS cancelled
FROM dst_project.flights AS f
WHERE f.status='Cancelled'
```
**Вопрос 2.**
Сколько самолетов моделей типа Boeing, Sukhoi Superjet, Airbus находится в базе авиаперевозок?
```sql
SELECT 'Boeing',
       count(ac.aircraft_code)
FROM dst_project.aircrafts ac
WHERE ac.model LIKE 'Boeing%'
UNION ALL
SELECT 'Sukhoi Superjet',
       count(ac.aircraft_code)
FROM dst_project.aircrafts ac
WHERE ac.model LIKE 'Sukhoi Superjet%'
UNION ALL
SELECT 'Airbus',
       count(ac.aircraft_code)
FROM dst_project.aircrafts ac
WHERE ac.model LIKE 'Airbus%'
```
**Вопрос 3.** 
В какой части (частях) света находится больше аэропортов?
```sql
SELECT 'Asia',
       count(ap.airport_code) AS airport_count
FROM dst_project.airports AS ap
WHERE ap.timezone LIKE 'Asia%'
UNION ALL
SELECT 'Europe',
       count(ap.airport_code)
FROM dst_project.airports AS ap
WHERE ap.timezone LIKE 'Europe%'
UNION ALL
SELECT 'Australia',
       count(ap.airport_code)
FROM dst_project.airports AS ap
WHERE ap.timezone LIKE 'Australia%'
ORDER BY 2 DESC
```
**Вопрос 4.**
У какого рейса была самая большая задержка прибытия за все время сбора данных? Введите id рейса (flight_id).
```sql
SELECT f.flight_id,
       (f.actual_arrival - f.scheduled_arrival) AS arrival_delta
FROM dst_project.flights f
WHERE f.actual_arrival IS NOT NULL
ORDER BY 2 DESC
LIMIT 1
```
### Задание 4.4
**Вопрос 1.**
Когда был запланирован самый первый вылет, сохраненный в базе данных?
```sql
SELECT to_char(min(f.scheduled_departure), 'DD.MM.YYYY')
FROM dst_project.flights f
```
**Вопрос 2.**
Сколько минут составляет запланированное время полета в самом длительном рейсе?
```sql
-- EPOCH extracts number of seconds
SELECT max(extract(epoch
                   FROM (f.scheduled_arrival - f.scheduled_departure)) / 60)
FROM dst_project.flights f
```
**Вопрос 3.**
Между какими аэропортами пролегает самый длительный по времени запланированный рейс?
```sql
SELECT f.departure_airport,
       f.arrival_airport,
       (f.scheduled_arrival - f.scheduled_departure) AS flight_len
FROM dst_project.flights f
ORDER BY 3 DESC
LIMIT 1
```
**Вопрос 4.**
Сколько составляет средняя дальность полета среди всех самолетов в минутах? Секунды округляются в меньшую сторону (отбрасываются до минут).
```sql
SELECT avg(extract(epoch
                   FROM (f.actual_arrival - f.actual_departure)) / 60)::int
FROM dst_project.flights f
WHERE f.actual_arrival IS NOT NULL
```
### Задание 4.5
**Вопрос 1.**
Мест какого класса у SU9 больше всего?
```sql
SELECT s.fare_conditions,
       count(*) AS seat_count
FROM dst_project.aircrafts ac
JOIN dst_project.seats s ON ac.aircraft_code=s.aircraft_code
WHERE ac.aircraft_code = 'SU9'
GROUP BY s.fare_conditions
ORDER BY 2 DESC
```
**Вопрос 2.**
Какую самую минимальную стоимость составило бронирование за всю историю?
```sql
SELECT min(b.total_amount)
FROM dst_project.bookings b
```
**Вопрос 3.**
Какой номер места был у пассажира с id = 4313 788533?
```sql
SELECT bp.seat_no
FROM dst_project.boarding_passes bp
JOIN dst_project.tickets t ON bp.ticket_no=t.ticket_no
WHERE t.passenger_id='4313 788533'
```
## Предварительный анализ
###  Задание 5.1
**Вопрос 1.**
Анапа — курортный город на юге России. Сколько рейсов прибыло в Анапу за 2017 год?
```sql
SELECT count(*)
FROM dst_project.flights f
JOIN dst_project.airports ap ON f.arrival_airport=ap.airport_code
WHERE ap.city='Anapa'
  AND f.actual_arrival IS NOT NULL
  AND date_part('year', f.actual_arrival)=2017
```
**Вопрос 2.**
Сколько рейсов из Анапы вылетело зимой 2017 года?
```sql
SELECT count(*)
FROM dst_project.flights f
JOIN dst_project.airports ap ON f.departure_airport=ap.airport_code
WHERE ap.city='Anapa'
  AND f.actual_departure IS NOT NULL
  AND (date_part('year', f.actual_departure)=2017
       AND date_part('month', f.actual_departure) IN (1,
                                                      2,
                                                      12))
```
**Вопрос 3.**
Посчитайте количество отмененных рейсов из Анапы за все время.
```sql
SELECT count(*)
FROM dst_project.flights f
JOIN dst_project.airports ap ON f.departure_airport=ap.airport_code
WHERE ap.city='Anapa'
  AND f.status='Cancelled'
```
**Вопрос 4.**
Сколько рейсов из Анапы не летают в Москву?
```sql
SELECT count(*)
FROM dst_project.flights f
JOIN dst_project.airports apd ON f.departure_airport=apd.airport_code
JOIN dst_project.airports apa ON f.arrival_airport=apa.airport_code
WHERE apd.city='Anapa'
  AND apa.city!='Moscow'
```
**Вопрос 5.**
Какая модель самолета летящего на рейсах из Анапы имеет больше всего мест?
```sql
SELECT ac.model,
       count(DISTINCT s.seat_no)
FROM dst_project.flights f
JOIN dst_project.aircrafts ac ON f.aircraft_code=ac.aircraft_code
JOIN dst_project.seats s ON f.aircraft_code=s.aircraft_code
JOIN dst_project.airports ap ON f.departure_airport=ap.airport_code
WHERE ap.city='Anapa'
GROUP BY 1
ORDER BY 2 DESC
```
## Запрос для выгрузки датасета.
*Примечание:* Запрос на выборку данных за период *"зима 2016-2017"*, т.е. за декабрь 2016 и январь-февраль 2017.
```sql
WITH flight_data AS
  (SELECT f.flight_id,
          f.flight_no,
          f.departure_airport,
          f.arrival_airport,
          f.aircraft_code,
          (extract(epoch
                   FROM f.scheduled_arrival) - extract(epoch
                                                       FROM f.scheduled_departure)) / 60 AS flight_length_min,
          to_char(f.scheduled_departure, 'YYYY-MM-DD') AS flight_departure_date,
          to_char(f.scheduled_departure, 'HH24:MI') AS flight_departure_time,
          to_char(f.scheduled_departure, 'D') AS flight_departure_weekday
   FROM dst_project.flights AS f
   WHERE f.departure_airport = 'AAQ'
     AND (date_trunc('month', f.scheduled_departure) in ('2016-12-01',
                                                         '2017-01-01',
                                                         '2017-02-01'))
     AND f.status NOT IN ('Cancelled') ),
     ticket_data AS
  (SELECT t.flight_id,
          count(t.ticket_no) AS flight_ticket_count,
          sum(t.amount) AS flight_ticket_sold_total
   FROM dst_project.ticket_flights AS t
   GROUP BY t.flight_id),
     aircraft_data AS
  (SELECT a.aircraft_code,
          a.model AS aircraft_model,
          count(s.seat_no) AS aircraft_seat_count
   FROM dst_project.aircrafts a
   JOIN dst_project.seats s ON a.aircraft_code = s.aircraft_code
   GROUP BY a.aircraft_code)
SELECT f.flight_id,
       f.flight_no,
       f.departure_airport,
       f.arrival_airport,
       f.flight_length_min,
       f.flight_departure_date,
       f.flight_departure_time,
       f.flight_departure_weekday,
       t.flight_ticket_count,
       t.flight_ticket_sold_total,
       a.aircraft_model,
       a.aircraft_seat_count,
       a.aircraft_code
FROM flight_data AS f
LEFT JOIN ticket_data AS t ON f.flight_id = t.flight_id
JOIN aircraft_data AS a ON f.aircraft_code = a.aircraft_code
ORDER BY f.flight_id,
         f.arrival_airport;
```
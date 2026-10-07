---
title: "PMDP — плитки Трамвай 1 и Автобус 30 в Home Assistant"
tags: [home-assistant, lovelace, pmdp, transport, plzen]
updated: 2026-10-07
---

# PMDP — плитки Трамвай 1 и Автобус 30 в Home Assistant

## Что сделано

Home Assistant получает ближайшие отправления напрямую из API PMDP (`jizdnirady.pmdp.cz`) без отдельного сервера и кастомной интеграции.

| Маршрут | Остановка | Направление | StopId | CISJRNumber | Опрос | Выборка |
|---|---|---|---:|---:|---:|---:|
| Трамвай 1 | Vřesová | Bolevec | 2 | 57187 | 240 с | 15 |
| Автобус 30 | Pošta Francouzská | Borská pole | 234 | 57077 | 120 с | 25 |

Для каждого направления один REST-запрос создаёт три сенсора: время отправления, минуты до отправления и задержку. Итого:

```text
sensor.tramvaj_1_vresova_bolevec
sensor.tramvaj_1_vresova_bolevec_leaving
sensor.tramvaj_1_vresova_bolevec_delay
sensor.autobus_30_posta_francouzska_borska_pole
sensor.autobus_30_posta_francouzska_borska_pole_leaving
sensor.autobus_30_posta_francouzska_borska_pole_delay
```

Проверено на Home Assistant 2026.9.4: все шесть сущностей существуют. Основной сенсор автобуса может временно возвращать `unknown`, если API не отдаёт подходящее ближайшее отправление.

## Какие конфиги изменили

Изменён файл:

```text
/config/configuration.yaml
```

В него добавлен раздел `rest:`. Если `rest:` уже существует, второй ключ создавать нельзя — элементы нужно добавить в существующий список.

Карточка Lovelace хранится отдельно. В текущих storage-дашбордах карточка с этими entity ID не найдена, поэтому YAML карточки ниже нужно сохранить через **Редактировать дашборд → Добавить карточку → Вручную**.

## REST-сенсоры в `configuration.yaml`

```yaml
rest:
  - resource: "https://jizdnirady.pmdp.cz/odjezdy/vyhledat"
    method: POST
    headers:
      Content-Type: "application/json"
      X-Requested-With: "XMLHttpRequest"
      User-Agent: "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36"
      Referer: "https://jizdnirady.pmdp.cz/odjezdy"
      Origin: "https://jizdnirady.pmdp.cz"
    payload: >-
      {"Stop":{"StopId":2,"CISJRNumber":57187},"DateAndTime":null,"MaxResults":15,"MaxResultsDateAndTime":null,"FullResults":true}
    scan_interval: 240
    sensor:
      - name: "Tramvaj 1 Vresova Bolevec"
        unique_id: pmdp_tram1_vresova_bolevec
        value_template: >
          {% set ns = namespace(dep=none) %}
          {% for item in value_json %}
            {% if item.Line.Name == '1' and item.LastStopName == 'Bolevec' and ns.dep == none %}
              {% set ns.dep = item.DepartureTime[11:16] %}
            {% endif %}
          {% endfor %}
          {{ ns.dep if ns.dep != none else 'unknown' }}
        icon: mdi:tram

      - name: "Tramvaj 1 Vresova Bolevec leaving"
        unique_id: pmdp_tram1_vresova_bolevec_leaving
        value_template: >
          {% set ns = namespace(leaving=none) %}
          {% for item in value_json %}
            {% if item.Line.Name == '1' and item.LastStopName == 'Bolevec' and ns.leaving == none %}
              {% set ns.leaving = item.LeavingInMin %}
            {% endif %}
          {% endfor %}
          {{ ns.leaving if ns.leaving != none else 'unknown' }}
        unit_of_measurement: "min"
        icon: mdi:clock-outline

      - name: "Tramvaj 1 Vresova Bolevec delay"
        unique_id: pmdp_tram1_vresova_bolevec_delay
        value_template: >
          {% set ns = namespace(delay=none) %}
          {% for item in value_json %}
            {% if item.Line.Name == '1' and item.LastStopName == 'Bolevec' and ns.delay == none %}
              {% set ns.delay = item.DelayMin if item.DelayMin is not none else 0 %}
            {% endif %}
          {% endfor %}
          {{ ns.delay if ns.delay != none else 0 }}
        unit_of_measurement: "min"
        icon: mdi:alert-circle-outline

  - resource: "https://jizdnirady.pmdp.cz/odjezdy/vyhledat"
    method: POST
    headers:
      Content-Type: "application/json"
      X-Requested-With: "XMLHttpRequest"
      User-Agent: "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36"
      Referer: "https://jizdnirady.pmdp.cz/odjezdy"
      Origin: "https://jizdnirady.pmdp.cz"
    payload: >-
      {"Stop":{"StopId":234,"CISJRNumber":57077},"DateAndTime":null,"MaxResults":25,"MaxResultsDateAndTime":null,"FullResults":true}
    scan_interval: 120
    sensor:
      - name: "Autobus 30 Posta Francouzska Borska pole"
        unique_id: pmdp_bus30_posta_borska
        value_template: >
          {% set ns = namespace(dep=none) %}
          {% for item in value_json %}
            {% if item.Line.Name == '30' and item.LastStopName == 'Borská pole' and ns.dep == none %}
              {% set ns.dep = item.DepartureTime[11:16] %}
            {% endif %}
          {% endfor %}
          {{ ns.dep if ns.dep != none else 'unknown' }}
        icon: mdi:bus

      - name: "Autobus 30 Posta Francouzska Borska pole leaving"
        unique_id: pmdp_bus30_posta_borska_leaving
        value_template: >
          {% set ns = namespace(leaving=none) %}
          {% for item in value_json %}
            {% if item.Line.Name == '30' and item.LastStopName == 'Borská pole' and ns.leaving == none %}
              {% set ns.leaving = item.LeavingInMin %}
            {% endif %}
          {% endfor %}
          {{ ns.leaving if ns.leaving != none else 'unknown' }}
        unit_of_measurement: "min"
        icon: mdi:clock-outline

      - name: "Autobus 30 Posta Francouzska Borska pole delay"
        unique_id: pmdp_bus30_posta_borska_delay
        value_template: >
          {% set ns = namespace(delay=none) %}
          {% for item in value_json %}
            {% if item.Line.Name == '30' and item.LastStopName == 'Borská pole' and ns.delay == none %}
              {% set ns.delay = item.DelayMin if item.DelayMin is not none else 0 %}
            {% endif %}
          {% endfor %}
          {{ ns.delay if ns.delay != none else 0 }}
        unit_of_measurement: "min"
        icon: mdi:alert-circle-outline
```

## Почему автобус настроен иначе

Автобус 30 ходит реже и при `MaxResults: 10` иногда не попадал в выдачу среди других маршрутов остановки. Поэтому для него `MaxResults` увеличен до `25`, а `scan_interval` уменьшен до 120 секунд. Для трамвая достаточно 15 результатов и 240 секунд.

`json_attributes_path` убран: chained-фильтры JSONPath работали нестабильно. Значения извлекаются отдельными `value_template`. PMDP иногда возвращает `DelayMin: null`, поэтому применяется явная проверка `is not none`, иначе сенсор задержки становится `unavailable`.

## Карточка Lovelace

```yaml
type: markdown
content: |
  ## 🚋 Tramvaj 1 → Bolevec
  **Zastávka:** Vřesová
  {% set dep = states('sensor.tramvaj_1_vresova_bolevec') %}
  {% set leaving = states('sensor.tramvaj_1_vresova_bolevec_leaving') %}
  {% set delay = states('sensor.tramvaj_1_vresova_bolevec_delay') %}
  {% if dep not in ['unknown', 'unavailable'] %}
  - **Odjezd:** {{ dep }}
  - **Za:** {{ leaving }} min
  - **Zpoždění:** {{ delay }} min
  {% else %}
  _Není dostupné_
  {% endif %}

  ---

  ## 🚌 Autobus 30 → Borská pole
  **Zastávka:** Pošta Francouzská
  {% set dep = states('sensor.autobus_30_posta_francouzska_borska_pole') %}
  {% set leaving = states('sensor.autobus_30_posta_francouzska_borska_pole_leaving') %}
  {% set delay = states('sensor.autobus_30_posta_francouzska_borska_pole_delay') %}
  {% if dep not in ['unknown', 'unavailable'] %}
  - **Odjezd:** {{ dep }}
  - **Za:** {{ leaving }} min
  - **Zpoždění:** {{ delay }} min
  {% else %}
  _Není dostupné_
  {% endif %}

  _Aktualizace karty: {{ now().strftime('%H:%M') }}_
```

`now()` показывает время перерисовки карточки, а не время последнего REST-запроса.

## Как применить

1. Сделать резервную копию `configuration.yaml`.
2. Добавить оба элемента в список `rest:`.
3. Выполнить **Настройки → Система → Проверка конфигурации**.
4. Перезапустить Home Assistant.
5. Проверить шесть сущностей в **Инструменты разработчика → Состояния**.
6. Добавить Markdown-карточку на нужный дашборд.

## Как настроить свои данные

### 1. Найти остановку

```bash
curl -G 'https://jizdnirady.pmdp.cz/zastavky/zastavky' \
  --data-urlencode 'filtr=Vřesová' \
  --data-urlencode 'maxVysledku=5'
```

Нужны `Placement.StopId` и `Placement.CISJRNumber`. Проверенные значения:

```text
Vřesová: StopId=2, CISJRNumber=57187
Pošta Francouzská: StopId=234, CISJRNumber=57077
```

### 2. Проверить отправления

```bash
curl -X POST 'https://jizdnirady.pmdp.cz/odjezdy/vyhledat' \
  -H 'Content-Type: application/json' \
  -H 'X-Requested-With: XMLHttpRequest' \
  -H 'Referer: https://jizdnirady.pmdp.cz/odjezdy' \
  -H 'Origin: https://jizdnirady.pmdp.cz' \
  -d '{"Stop":{"StopId":2,"CISJRNumber":57187},"DateAndTime":null,"MaxResults":15,"MaxResultsDateAndTime":null,"FullResults":true}'
```

Поля ответа:

| Поле | Назначение |
|---|---|
| `Line.Name` | номер маршрута |
| `LastStopName` | конечная/направление |
| `DepartureTime` | время отправления |
| `LeavingInMin` | минут до отправления |
| `DelayMin` | задержка; иногда `null` |

### 3. Скопировать REST-блок и заменить

- `StopId` и `CISJRNumber` в `payload`;
- маршрут в `item.Line.Name == '...'`;
- направление в `item.LastStopName == '...'`;
- `name`, `unique_id` и entity ID в карточке;
- иконку (`mdi:tram`, `mdi:bus`, `mdi:trolleybus`);
- при редком маршруте увеличить `MaxResults`.

`DateAndTime: null` запрашивает ближайшие отправления относительно текущего времени; календарь рабочего/выходного дня учитывает сервер PMDP.

## Диагностика

### Основной сенсор показывает `unknown`

- проверить точное написание `Line.Name` и `LastStopName`, включая диакритику;
- выполнить POST-запрос вручную;
- увеличить `MaxResults`;
- проверить, есть ли рейсы по направлению в текущее время.

### Задержка показывает `unavailable`

Проверить обработку `DelayMin: null` и использовать `is not none`.

### Сенсоры не обновляются

- проверить доступ Home Assistant к `https://jizdnirady.pmdp.cz`;
- проверить журнал интеграции `rest`;
- проверить YAML и уровень `scan_interval`.

### Карточка не появилась после перезапуска

Перезапуск создаёт сенсоры, но не добавляет Lovelace-карточку. Её нужно сохранить отдельно в редакторе дашборда.

## Дополнительно: карта движения

```yaml
type: iframe
url: https://jizdnirady.pmdp.cz/provoz
aspect_ratio: 75%
```

Работа iframe зависит от текущих заголовков безопасности сайта PMDP.

## Источники

- <https://jizdnirady.pmdp.cz/>
- <https://jizdnirady.pmdp.cz/provoz>
- История настройки: @session:default/20260622_081348_1a73de78

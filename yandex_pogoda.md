Погодный информер для Inker использующий в качестве источника интеграцию для Home Assistant, Yandex Pogoda.
Данные из HA в Inker передаются посредством двух источников(Data Source):
1) Непосредственно из интеграции Yandex Pogoda, Data Source Connection URL `http://homeassistant.loc:8123/api/states/weather.yandex_pogoda`
2) и вспомогательного датчика Eink Weather Widget, Data Source Connection URL `http://homeassistant.loc:8123/api/states/sensor.eink_weather_widget`

Код для вспомогательного датчика(packages) Eink Weather Widget:
```yaml
template:
  - sensor:
      - name: "Eink Weather Widget"
        unique_id: eink_weather_widget
        state: "{{ states('weather.yandex_pogoda') }}"
        attributes:
          temperature: "{{ state_attr('weather.yandex_pogoda', 'temperature') }}"
          apparent_temperature: "{{ state_attr('weather.yandex_pogoda', 'apparent_temperature') }}"
          yandex_condition: "{{ state_attr('weather.yandex_pogoda', 'yandex_condition') }}"
          forecast_hourly: "{{ state_attr('weather.yandex_pogoda', 'forecast_hourly') | to_json }}"
          next_rising: "{{ state_attr('sun.sun', 'next_rising') }}"
          next_setting: "{{ state_attr('sun.sun', 'next_setting') }}"
```
Перед установкой виджета в Inker проверить в HA в разделе Инструментарий > Состояние наличие `weather.yandex_pogoda` и `sensor.eink_weather_widget`
Для установки в Inker можно воспользоваться установочным кодом
После установки нкжно будетвнести изменеия в источниках в пунктах Custom Headers (Add headers for authentication or custom requirements) добавить свои данные Токен для подключения к HA.

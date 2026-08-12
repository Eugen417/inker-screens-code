# Виджеты для Inker

1) 6 виджетов, Погодный информер: [Yandex Pogoda](/yandex_pogoda/yandex_pogoda.md)
   
      <img width="250" height="100%" alt="Yandex Pogoda" src="https://github.com/user-attachments/assets/36f4277e-b30e-4a61-95c0-98ebb6c1cba8" /> 


2) 6 виджетов, информер [Космическая погода](/space_weather/space_weather_xras.md) по данным [xras.ru](https://xras.ru)
   
      <img width="250" height="100%" alt="Space Weather" src="https://github.com/user-attachments/assets/72b09ea2-d27b-4993-859e-0f5261b57f13" />


3) 3 Виджета [Информера о луне](/space_weather/moon.md)

      <img width="250" height="100%" alt="Мoon" src="https://github.com/user-attachments/assets/f7f14482-5e72-4095-8598-9dced36560e8" />



4) 15 виджетов, Погодный информер: [Open Meteo](https://github.com/Eugen417/inker-screens-code/blob/main/open_meteo.md)

---

# Файл конфигурации ESPHome Device Builder для проекта: [Дисплей E-Ink | ESPHome-TRMNL 7,5"](https://3dpm.ru/en/open_projects/ESPHome-TRMNL_7+5/):


1. **Проблема с выходом из глубокого сна:** Когда дисплей спит, нажатие кнопки физически будит плату. К моменту загрузки прошивки кнопка *уже* находится в зажатом состоянии. Компонент `on_multi_click` ожидает перехода из отпущенного состояния в нажатое, поэтому таймер для перехода в режим настройки даже не запускается.
2. **Гонка процессов при старте (Race condition):** Плата подключается к Wi-Fi и успевает обновить экран с последующим уходом в сон быстрее, чем вы отпустите кнопку.
3. **Рассинхронизация документации и кода:** В тексте правил заявлено долгое нажатие **более 5 секунд**, а в вашем блоке `binary_sensor` прописано удержание всего на 2 секунды (`ON for at least 2s`).


4. **Отсутствие отрисовки при активном Wi-Fi:** В текущем коде при входе в режим настройки (OTA) в активном режиме проверяется только статус точки доступа (Captive Portal). Если плата уже подключена к домашнему Wi-Fi, устройство не спит 30 минут, однако экран со ссылкой на web-интерфейс просто не прорисовывается.



---

### Как это исправить

Вам нужно заменить три блока в вашем YAML-конфиге на исправленные версии ниже. Эти исправления устраняют гонку процессов, синхронизируют тайминги с документацией (5 секунд) и гарантируют правильный вывод QR-кодов.

#### 1. Обновление блока `esphome` (Обучение платы понимать долгое нажатие из сна)

Замените всю секцию `on_boot` внутри `esphome:`. Теперь плата после загрузки будет корректно отсчитывать 5 секунд для входа в OTA-режим.

```yaml
esphome:
  min_version: 2026.1.3 
  name: $name
  friendly_name: $friendly_name
  on_boot:
    priority: -10
    then:
      - delay: 100ms 
      - if:
          condition:
            lambda: 'return id(back_button).state;' 
          then:
            - logger.log: "Кнопка зажата при старте! Взведен флаг очистки Screen Design ID Override."
            - lambda: 'id(clear_override_flag) = true;'
            # Ожидаем еще 4.9 сек для проверки на 5-секундное удержание (Режим настройки)
            - delay: 4900ms
            - if:
                condition:
                  lambda: 'return id(back_button).state;'
                then:
                  - logger.log: "Кнопка удерживалась 5 секунд! Активирован режим OTA."
                  - lambda: 'id(ota_mod) = true;'
                  
      # Проверка наличия сохраненных данных Wi-Fi
      - if:
          condition:
            lambda: 'return !wifi::global_wifi_component->has_sta();'
          then:
            - logger.log: "No saved Wi-Fi credentials! Entering AP setup mode."
            - lambda: 'id(ota_mod) = true;'
            - wait_until:
                condition: wifi.ap_active
                timeout: 15s
            - script.execute: generate_qr_link_ap
            - delay: 16ms
            - script.execute:
                id: safe_update_display
                target_page: 5
                force_update: true

      # Продолжаем обычный запуск, если режим OTA не активен
      - if:
          condition:
            lambda: 'return !id(ota_mod);'
          then:
            - delay: $wifi_connect_delay
            - if:
                condition:
                  not:
                    wifi.connected:
                then:
                  - script.execute:
                      id: trmnl_error
                      error_msg: "Wi-Fi timeout"

```

#### 2. Обновление блока `wifi` (Устранение засыпания во время удержания)

В блок `on_connect` добавлена пауза (`wait_until`), которая не дает плате скачивать картинку и уходить в сон, пока вы не отпустите кнопку (либо пока не пройдет 5 секунд). Полностью замените блок `wifi:`:

```yaml
wifi:
  ssid: $wifi_ssid
  password: $wifi_password
  min_auth_mode: WPA2
  power_save_mode: LIGHT
  fast_connect: true 
  output_power: $wifi_output_power
  ap:
    ap_timeout: 15s
    password: $wifi_ap_password
  on_connect:
    then:
      # --- ФИКС ГОНКИ ПРИ ЗАГРУЗКЕ ---
      # Ждем отпускания кнопки, либо включения ota_mod из on_boot (максимум 6 сек)
      - wait_until:
          condition:
            or:
              - lambda: 'return !id(back_button).state;'
              - lambda: 'return id(ota_mod);'
          timeout: 6s
      # -------------------------------
      - if:
          condition:
            lambda: 'return !id(ota_mod);'
          then:
            # ПРОВЕРКА ЗАРЯДА АККУМУЛЯТОРА
            - if:
                condition:
                  and:
                    - lambda: 'return id(low_battery_threshold).state > 0.0;' 
                    - lambda: 'return id(battery_voltage).state > 0.0 && id(battery_voltage).state <= id(low_battery_threshold).state;' 
                then:
                  - logger.log: "Напряжение аккумулятора ниже установленного порога! Вывод экрана оповещения."
                  - script.execute:
                      id: safe_update_display
                      target_page: 6 
                      force_update: true
                  - delay: $deep_sleep_delay
                  - deep_sleep.enter:
                      sleep_duration: 49 d
                else:
                  - script.execute: trmnl_config 
          else:
            - logger.log: "Wi-Fi подключен, активен OTA MODE"
            - delay: 16ms
            - script.execute: generate_qr_link_web_server
            - delay: 16ms
            - script.execute:
                id: safe_update_display
                target_page: 4 
                force_update: true

```

#### 3. Обновление блока `binary_sensor` (Исправление кнопки в активном режиме)

Замените ваш `binary_sensor`. Теперь время удержания синхронизировано с правилами (5 секунд), а плата корректно выводит экран с QR-кодом независимо от того, подключена она к вашему Wi-Fi, или перешла в режим AP (Captive Portal).

```yaml
binary_sensor:
  - platform: gpio
    name: Button
    id: back_button
    pin:
      number: GPIO32
      mode:
        input: true
        pullup: true
      allow_other_uses: true  
      inverted: true
    internal: True
    filters:
      - delayed_on_off: 20ms
    on_multi_click:
      # 1. Удержание кнопки более 5 секунд — Запуск OTA-режима (в активном состоянии)
      - timing:
          - ON for at least 5s
        then:
          - logger.log: "Удержание кнопки 5 сек! Вход в режим OTA."
          - lambda: 'id(ota_mod) = true;'
          - if:
              condition: wifi.connected
              then:
                - script.execute: generate_qr_link_web_server
                - script.execute:
                    id: safe_update_display
                    target_page: 4 
                    force_update: true
              else:
                - wait_until:
                    condition: wifi.ap_active
                    timeout: 20s
                - if:
                    condition: wifi.ap_active
                    then:
                      - script.execute: generate_qr_link_ap
                      - script.execute:
                          id: safe_update_display
                          target_page: 5 
                          force_update: true

      # 2. Короткое нажатие — Перезагрузка
      - timing:
          - ON for at most 1s
          - OFF for at least 0.5s
        then:
          - button.press: restart_button

```



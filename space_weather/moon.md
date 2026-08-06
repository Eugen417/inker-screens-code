<img width="500" height="100%" alt="Снимок экрана — 2026-08-06 в 21 19 26" src="https://github.com/user-attachments/assets/43306658-0113-4df0-8fb5-a6c3d643a888" />


Требования:
Установленная и настроенная интеграция [Lunar Phase Integration for Home Assistant](https://github.com/ngocjohn/lunar-phase)
Установлен пользовательский датчик
Вспомогательный сенсор собирает все нужные массивы прогнозов и состояний в один пакет. Добавьте следующий код в вашу конфигурацию HA (например, в файл `configuration.yaml` или в папку `packages`):

```yaml
template:
  - sensor:
      - name: "Eink Full Space Weather"
        unique_id: eink_full_space_weather
        state: "{{ states('sensor.moscow_kp_current') }}"
        attributes:
          # --- 1. АВРОРА (ПОЛЯРНОЕ СИЯНИЕ) ---
          aurora_index: "{{ states('sensor.moscow_aurora_index_latest') }}"
          aurora_prob: "{{ states('sensor.moscow_aurora_probability_local') }}"
          aurora_history: "{{ state_attr('sensor.moscow_aurora_probability_local', 'history') | to_json }}"
          
          # --- 2. KP-ИНДЕКСЫ И СОЛНЕЧНАЯ АКТИВНОСТЬ ---
          kp_current: "{{ states('sensor.moscow_kp_current') }}"
          kp_today: "{{ states('sensor.moscow_kp_forecast_today') }}"
          kp_tomorrow: "{{ states('sensor.moscow_kp_forecast_tomorrow') }}"
          forecast27_kp: "{{ states('sensor.moscow_forecast27_max_kp') }}"
          f10_today: "{{ states('sensor.moscow_f10_forecast_today') }}"
          
          # --- 3. ВЕТЕР, ИЗЛУЧЕНИЕ И ВСПЫШКИ ---
          swv_current: "{{ states('sensor.moscow_swv_current') }}"
          xray_current: "{{ states('sensor.moscow_xray_current') }}"
          flare_status: "{{ states('sensor.moscow_solar_flare_current_status') }}"
          flare_info: "{{ states('sensor.moscow_solar_flare_last_info') }}"
          
          # --- 4. МАССИВЫ ПРОГНОЗОВ (обязательно с to_json) ---
          forecast27_array: >
            {{ state_attr('sensor.moscow_forecast27_max_kp', 'forecast_array') | to_json }}
          forecast_3d_array: >
            {{ state_attr('sensor.moscow_xras_storm_probability', 'forecast_3d_array') | to_json }}
          kp_forecast_3h: >
            {{ state_attr('sensor.moscow_kp_current', 'forecast_3h') | to_json }}
          kp_forecast_daily: >
            {{ state_attr('sensor.moscow_kp_forecast_today', 'forecast_daily') | to_json }}
          history_3d: >
            {{ state_attr('sensor.moscow_kp_current', 'history_3d') | to_json }}

          # --- 5. ЛУНА (БАЗОВЫЕ ПАРАМЕТРЫ) ---
          moon_phase: "{{ states('sensor.moscow_moon_phase') }}"
          moon_illumination: "{{ states('sensor.moscow_osveshchennost_luny') }}"
          moon_age: "{{ states('sensor.moscow_vozrast_luny') }}"
          moon_distance: "{{ states('sensor.moscow_rasstoianie_do_luny') }}"
          moon_azimuth: "{{ states('sensor.moscow_azimut_luny') }}"
          moon_altitude: "{{ states('sensor.moscow_vysota_luny') }}"
          moon_parallactic: "{{ states('sensor.moscow_parallakticheskii_ugol_luny') }}"
          
          # --- 6. ЛУНА (ТАЙМИНГИ И ФАЗЫ) ---
          moon_rise: "{{ states('sensor.moscow_voskhod_luny') }}"
          moon_set: "{{ states('sensor.moscow_zakhod_luny') }}"
          moon_high: "{{ states('sensor.moscow_luna_v_zenite') }}"
          next_moon_phase: "{{ states('sensor.moscow_sleduiushchaia_faza_luny') }}"
          next_new_moon: "{{ states('sensor.moscow_sleduiushchee_novolunie') }}"
          next_full_moon: "{{ states('sensor.moscow_sleduiushchee_polnolunie') }}"
          next_first_quarter: "{{ states('sensor.moscow_sleduiushchaia_pervaia_chetvert') }}"
          next_third_quarter: "{{ states('sensor.moscow_sleduiushchaia_tretia_chetvert') }}"
          
          # --- 7. ДАННЫЕ ДЛЯ ТАБЛИЦ И ПРОГНОЗОВ (ДИНАМИЧЕСКИЕ) ---
          flare_summary: "{{ state_attr('sensor.moscow_solar_flare_current_status', 'flare_summary') }}"
          flare_index: "{{ state_attr('sensor.moscow_solar_flare_current_status', 'flare_index') }}"
          flares_list: >
            {{ state_attr('sensor.moscow_solar_flare_current_status', 'flares_list') | to_json }}
          storm_prob_today: >
            {{ state_attr('sensor.moscow_xras_storm_probability', 'prob_today') | to_json }}
          storm_prob_tomorrow: >
            {{ state_attr('sensor.moscow_xras_storm_probability', 'prob_tomorrow') | to_json }}

          # --- 8. СОЛНЕЧНЫЕ ПЯТНА ---
          sunspots_total_groups: "{{ state_attr('sensor.moscow_f10_forecast_today', 'sunspots_total_groups') }}"
          sunspots_total_area: "{{ state_attr('sensor.moscow_f10_forecast_today', 'sunspots_total_area') }}"
          sunspots_list: >
            {{ state_attr('sensor.moscow_f10_forecast_today', 'sunspots_list') | to_json }}
          
          # --- 9. ПАРАМЕТРЫ СОЛНЕЧНОГО ВЕТРА ---
          sw_density: "{{ state_attr('sensor.moscow_swv_current', 'sw_density') }}"
          sw_temp: "{{ state_attr('sensor.moscow_swv_current', 'sw_temp') }}"
          sw_bt: "{{ state_attr('sensor.moscow_swv_current', 'sw_bt') }}"
          sw_bz: "{{ state_attr('sensor.moscow_swv_current', 'sw_bz') }}"
          
          # --- 10. ГРАФИКИ ВЕТРА (за час) ---
          sw_history_v: >
            {{ state_attr('sensor.moscow_swv_current', 'history') | to_json }}
          sw_history_n: >
            {{ state_attr('sensor.moscow_swv_current', 'history_density') | to_json }}
          sw_history_t: >
            {{ state_attr('sensor.moscow_swv_current', 'history_temp') | to_json }}
          sw_history_bt: >
            {{ state_attr('sensor.moscow_swv_current', 'history_bt') | to_json }}
          sw_history_bz: >
            {{ state_attr('sensor.moscow_swv_current', 'history_bz') | to_json }}

```

# Установка
## Способ А: Быстрый импорт (Screen Code):
1. В Inker перейдите в раздел **Screens > Import**.
2. Откройте файл [sw_inker_screen.json](moon_inker_screen.json) в этом репозитории, скопируйте его содержимое целиком и нажмите **Import Screen**.
3. После импорта обязательно зайдите в раздел Data Sources, внесите изменения в:
*   **Basic Information**
    *   **Name:** `HA Space Weather`
*   **Connection**
    *   **URL:** `http://homeassistant.local:8123/api/states/sensor.eink_full_space_weather` проверьте ссылку
*   **Custom Headers** *(Add headers for authentication)*
    *   **Header name:** `Authorization`
    *   **Header value:** `Bearer ВАШ_ТОКЕН_HA`

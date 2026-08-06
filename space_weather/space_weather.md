Отличная идея! Документация в едином стиле поможет вам (и тем, с кем вы решите поделиться) легко переносить этот дашборд на другие устройства или восстанавливать после сброса системы.

Вот полный Markdown-файл (инструкция) для вашего дашборда Космической Погоды по образу и подобию погодного.

---

# Информер Космической погоды для Inker (Home Assistant)

Краткое руководство по настройке информера космической погоды (магнитные бури, полярные сияния, солнечные вспышки и лунный радар) для E-ink дисплеев Inker с использованием данных из Home Assistant.

Для работы виджетов у вас должны быть установлены необходимые интеграции (Space Weather, Moon и др.), отдающие базовые сенсоры (xras.ru, aurora и т.д.). Ниже приведена процедура создания единого вспомогательного сенсора, который объединит все данные для отправки на экран.

### Настройка Home Assistant (Создание сенсора)

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

> **Проверка:** После перезагрузки шаблонов (или перезапуска HA) зайдите в *Панель разработчика > Состояния* и убедитесь, что `sensor.eink_full_space_weather` существует и содержит все массивы в формате JSON (например, `forecast_3d_array` не равен null).

---

## 1. Источник данных (Data Source)

Для всех виджетов этого экрана используется один универсальный источник данных. В Inker перейдите в **Data Sources** и создайте его:

* **Basic Information**
* **Name:** `HA Space Weather`
* **Type:** `JSON API`


* **Connection**
* **URL:** `[http://homeassistant.local:8123/api/states/sensor.eink_full_space_weather](http://homeassistant.local:8123/api/states/sensor.eink_full_space_weather)` (Убедитесь, что IP или домен правильный)
* **HTTP Method:** `GET`


* **Custom Headers** *(Для аутентификации в HA)*
* **Header name:** `Authorization`
* **Header value:** `Bearer ВАШ_ТОКЕН_HA` (Сгенерируйте долгоживущий токен доступа в профиле HA)



---

## 2. Библиотека виджетов

Для каждого виджета из списка ниже применяются **общие базовые настройки**:

* **Data Source:** `HA Space Weather`
* **Choose Display Type:** `Grid`
* **Field:** `entity_id` *(Для наших скриптов поле значения не имеет, но обязательно для заполнения в Inker)*
* **Display As:** `Image` *(Обязательно для корректного отображения SVG!)*
* **Grid Settings:** Columns: `1`, Rows: `1`

Перейдите в настройки ячейки (**Cell 0, 0**) и вставьте соответствующий код в блок **JavaScript Code**.

---

### 📌 Виджет 1: Магнитные бури (График на 24 часа)

Отображает 8 трехчасовых столбиков за последние сутки с умным округлением и выделением магнитных бурь.

```javascript
try {
  let src = typeof $ === 'string' ? JSON.parse($) : $;
  let attr = src.attributes || {};
  
  let kpCurrent = parseFloat(attr.kp_current || 0);

  let formatKp = (v) => {
    let base = Math.round(v), diff = v - base;
    if (Math.abs(diff) < 0.2) return base.toString(); 
    return diff > 0 ? base + "+" : base + "-";
  };

  let history3d = [];
  try { history3d = typeof attr.history_3d === 'string' ? JSON.parse(attr.history_3d) : attr.history_3d; } catch(e) {}
  
  let flatData = [];
  if (history3d && history3d.length > 0) {
      history3d.sort((a, b) => a.time.localeCompare(b.time));
      const hoursList = ['h00', 'h03', 'h06', 'h09', 'h12', 'h15', 'h18', 'h21'];
      history3d.forEach(day => {
          hoursList.forEach(hour => {
              if (day[hour] !== null && day[hour] !== 'null' && day[hour] !== undefined) {
                  flatData.push({ val: parseFloat(day[hour]), startHour: parseInt(hour.substring(1)) });
              }
          });
      });
  }
  
  let barsData = flatData.slice(-8);
  while (barsData.length < 8) {
      let lastStart = barsData.length > 0 ? barsData[0].startHour : 3;
      barsData.unshift({ val: 0, startHour: (lastStart - 3 + 24) % 24 });
  }
  
  if (attr.kp_current !== undefined && attr.kp_current !== null) barsData[7].val = kpCurrent; 

  let maxKp = Math.max(...barsData.map(b => b.val));
  if (isNaN(maxKp)) maxKp = 0;
  
  let statusText = "Спокойная обстановка";
  if (maxKp >= 4 && maxKp < 5) statusText = "Возмущенное поле";
  else if (maxKp >= 5) statusText = "Магнитная буря";

  let pBarWidth = Math.max(Math.min((maxKp / 9) * 236, 236), 0);
  if (maxKp > 0 && pBarWidth < 6) pBarWidth = 6;

  let timeLabels = [];
  for(let i = 0; i < 8; i++) timeLabels.push(String(barsData[i].startHour).padStart(2, '0'));
  let lastHour = barsData[7].startHour;
  if(isNaN(lastHour)) lastHour = 0;
  timeLabels.push(String((lastHour + 3) % 24).padStart(2, '0')); 

  let barsSvg = "", valuesSvg = "", axisSvg = ""; 
  let barWidth = 18, gap = 12, startX = 20, baseY = 193, maxH = 45;

  let xFirst = startX - gap/2;
  let xLast = startX - gap/2 + 8 * (barWidth + gap);
  axisSvg += `<line x1="${xFirst}" y1="${baseY}" x2="${xLast}" y2="${baseY}" stroke="#000" stroke-width="1" stroke-linecap="round"/>`;
  
  for(let i = 0; i <= 8; i++) {
      let bx = startX - gap/2 + i * (barWidth + gap);
      axisSvg += `<text x="${bx}" y="${baseY + 12}" font-size="10" fill="#000" text-anchor="middle" font-family="Arial">${timeLabels[i]}</text>`;
  }

  barsData.forEach((item, i) => {
    let val = item.val, h = Math.max((val / 9) * maxH, 3); 
    let x = startX + i * (barWidth + gap), y = baseY - h;
    let cx = x + barWidth/2; 
    
    let fill = "none";              
    if (val >= 5) fill = "#000";    
    else if (val >= 4) fill = "#999"; 

    barsSvg += `<rect x="${x}" y="${y}" width="${barWidth}" height="${h}" fill="${fill}" stroke="#000" stroke-width="2" rx="3"/>`;
    
    let displayVal = "-";
    if (val !== undefined && val !== null && !isNaN(val)) {
        displayVal = formatKp(val);
        if (displayVal === "0" && val === 0) displayVal = "0"; 
    }

    valuesSvg += `<text x="${cx}" y="142" font-size="13" fill="#000" text-anchor="middle" font-family="Arial" font-weight="bold">${displayVal}</text>`;

    if (i === 7) {
        valuesSvg += `<text x="${cx}" y="118" font-size="10" fill="#000" text-anchor="middle" font-family="Arial">сейч</text>`;
        valuesSvg += `<line x1="${cx - 10}" y1="123" x2="${cx + 10}" y2="123" stroke="#000" stroke-width="1.5"/>`;
    }
  });

  let svg = `
  <svg xmlns="http://www.w3.org/2000/svg" width="260" height="210" viewBox="0 0 260 210">
    <style>.t { font-family: Arial, sans-serif; fill: #000; font-weight: bold; } .s { font-family: Arial, sans-serif; fill: #555; font-weight: bold; }</style>
    <text x="10" y="20" font-size="16" class="t" letter-spacing="1">МАГНИТНЫЕ БУРИ 24 Ч</text>
    <line x1="10" y1="28" x2="250" y2="28" stroke="#000" stroke-width="3" stroke-linecap="round"/>
    <text x="10" y="64" font-size="42" class="t">Kp ${formatKp(maxKp)}</text>
    <text x="10" y="82" font-size="14" class="t">${statusText}</text>
    <rect x="10" y="94" width="240" height="14" rx="7" fill="none" stroke="#000" stroke-width="2"/>
    <rect x="12" y="96" width="${pBarWidth}" height="10" rx="5" fill="#000"/>
    ${axisSvg}${valuesSvg}${barsSvg}
  </svg>`;
  
  return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(svg.replace(/\n\s*/g, ''));
} catch (err) { return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(`<svg width="260" height="210"><text y="20">Ошибка: ${err.message}</text></svg>`); }

```

---

### 📌 Виджет 2: Прогноз магнитных бурь на 2 дня

Показывает ожидаемый максимальный индекс (Kp) на ближайшие 24 часа вперед.

```javascript
try {
  let src = typeof $ === 'string' ? JSON.parse($) : $;
  let attr = src.attributes || {};
  
  let now = new Date();
  let currentTs = now.getTime();
  let endTs = currentTs + (24 * 60 * 60 * 1000); 
  
  let maxKp24 = 0;
  let forecastArr = [];
  try {
      if (typeof attr.forecast_3d_array === 'string') forecastArr = JSON.parse(attr.forecast_3d_array);
      else if (Array.isArray(attr.forecast_3d_array)) forecastArr = attr.forecast_3d_array;
  } catch(e) {}
  
  if (forecastArr && forecastArr.length > 0) {
    let futureVals = [];
    const hourKeys = ['h00', 'h03', 'h06', 'h09', 'h12', 'h15', 'h18', 'h21'];
    
    forecastArr.forEach(day => {
      let dayStr = day.time; 
      hourKeys.forEach(hk => {
         let blockTime = new Date(`${dayStr}T${hk.replace('h', '')}:00:00`);
         let blockTs = blockTime.getTime();
         if (blockTs > currentTs - (3 * 3600 * 1000) && blockTs <= endTs) {
            let val = parseFloat(day[hk]);
            if (!isNaN(val)) futureVals.push(val);
         }
      });
    });
    
    if (futureVals.length > 0) maxKp24 = Math.max(...futureVals);
    else maxKp24 = parseFloat(attr.kp_today || 0); 
  } else {
     maxKp24 = parseFloat(attr.kp_today || 0);
  }
  
  let formatKp = (v) => {
    let base = Math.round(v), diff = v - base;
    if (Math.abs(diff) < 0.2) return base.toString(); 
    return diff > 0 ? base + "+" : base + "-";
  };

  let d = new Date();
  const months = ['января', 'февраля', 'марта', 'апреля', 'мая', 'июня', 'июля', 'августа', 'сентября', 'октября', 'ноября', 'декабря'];
  let date1 = d.getDate() + " " + months[d.getMonth()];
  d.setDate(d.getDate() + 1);
  let date2 = d.getDate() + " " + months[d.getMonth()];

  let parseProb = (val, defaultArr) => {
      if (!val || val === "null" || val === "unknown") return defaultArr;
      if (Array.isArray(val)) return val;
      try {
          let parsed = typeof val === 'string' ? JSON.parse(val) : val;
          if (typeof parsed === 'string') parsed = JSON.parse(parsed);
          return (Array.isArray(parsed) && parsed.length >= 3) ? parsed : defaultArr;
      } catch(e) { return defaultArr; }
  };

  let p1 = parseProb(attr.prob_today || attr.storm_prob_today, [0, 0, 0]); 
  let p2 = parseProb(attr.prob_tomorrow || attr.storm_prob_tomorrow, [0, 0, 0]); 

  let drawBars = (x, data, label) => {
    let h0 = Math.max(((data[0] || 0) / 100) * 40, 4);
    let h1 = Math.max(((data[1] || 0) / 100) * 40, 4);
    let h2 = Math.max(((data[2] || 0) / 100) * 40, 4);
    let baseY = 188;

    return `
      <text x="${x-22}" y="135" font-size="13" class="t" text-anchor="middle">${data[0] || 0}</text>
      <text x="${x}" y="135" font-size="13" class="t" text-anchor="middle">${data[1] || 0}</text>
      <text x="${x+22}" y="135" font-size="13" class="t" text-anchor="middle">${data[2] || 0}</text>
      
      <rect x="${x-30}" y="${baseY - h0}" width="16" height="${h0}" fill="none" stroke="#000" stroke-width="2" rx="3"/>
      <rect x="${x-8}" y="${baseY - h1}" width="16" height="${h1}" fill="#999" stroke="#000" stroke-width="2" rx="3"/>
      <rect x="${x+14}" y="${baseY - h2}" width="16" height="${h2}" fill="#000" stroke="#000" stroke-width="2" rx="3"/>
      
      <line x1="${x-38}" y1="${baseY + 6}" x2="${x+38}" y2="${baseY + 6}" stroke="#000" stroke-width="2" stroke-linecap="round"/>
      <text x="${x}" y="${baseY + 18}" font-size="12" class="t" text-anchor="middle">${label}</text>
    `;
  };

  let svg = `
  <svg xmlns="http://www.w3.org/2000/svg" width="260" height="210" viewBox="0 0 260 210">
    <style>.t { font-family: Arial, sans-serif; fill: #000; font-weight: bold; }</style>
    <text x="10" y="20" font-size="15" class="t" letter-spacing="1">ПРОГНОЗ БУРЬ НА 2 ДН.</text>
    <line x1="10" y1="28" x2="250" y2="28" stroke="#000" stroke-width="3" stroke-linecap="round"/>
    <text x="10" y="75" font-size="46" class="t">Kp ${formatKp(maxKp24)}</text>
    <text x="10" y="95" font-size="12" class="t">Макс. на 24 ч вперёд</text>
    <text x="130" y="115" font-size="11" class="t" text-anchor="middle">ВЕРОЯТНОСТИ БУРЬ, %</text>
    ${drawBars(65, p1, date1)}
    ${drawBars(195, p2, date2)}
  </svg>`;
  
  return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(svg.replace(/\n\s*/g, ''));
} catch (err) { return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(`<svg width="260" height="210"><text y="20">Ошибка: ${err.message}</text></svg>`); }

```

---

### 📌 Виджет 3: Полярные сияния (Линейный график)

Отображает вероятность и индекс АИ (Северного полушария) в виде динамического линейного графика за 24 часа.

```javascript
try {
  let src = typeof $ === 'string' ? JSON.parse($) : $;
  let attr = src.attributes || {};

  let auroraProb = parseFloat(attr.aurora_prob || 0).toFixed(0);
  let auroraIndex = parseFloat(attr.aurora_index || 0).toFixed(1);

  let statusText = "Сияний не ожидается";
  if (auroraProb > 10 && auroraProb <= 30) statusText = "Слабая вероятность";
  else if (auroraProb > 30 && auroraProb <= 60) statusText = "Умеренная вероятность";
  else if (auroraProb > 60) statusText = "Высокая вероятность!";

  let pBarWidth = Math.max(Math.min((auroraProb / 100) * 110, 110), 0);
  let iBarWidth = Math.max(Math.min((auroraIndex / 10) * 110, 110), 0);

  let chartHtml = "";
  let aurHist = [];
  try {
      let raw = attr.aurora_history;
      if (typeof raw === 'string') raw = JSON.parse(raw);
      if (typeof raw === 'string') raw = JSON.parse(raw);
      if (Array.isArray(raw)) aurHist = raw;
  } catch(e) {}

  if (aurHist.length > 0) {
      aurHist.sort((a, b) => new Date(a.time) - new Date(b.time));
      
      let newestTs = new Date(aurHist[aurHist.length - 1].time).getTime();
      let cutoffTs = newestTs - (24 * 60 * 60 * 1000); 
      aurHist = aurHist.filter(pt => new Date(pt.time).getTime() >= cutoffTs);

      let maxVal = 0;
      aurHist.forEach(pt => {
          let v = parseFloat(pt.n_ai);
          if (!isNaN(v) && v > maxVal) maxVal = v;
      });
      
      let maxAi = Math.max(2, Math.ceil(maxVal));
      if (maxAi - maxVal < 0.2) maxAi += 1; 

      let chartX = 10;
      let chartY = 205; 
      let chartW = 225; 
      let chartH = 45;  

      if (maxAi >= 5) {
          let dangerY = chartY - (5 / maxAi) * chartH;
          let topDangerH = chartH - (chartY - dangerY); 
          chartHtml += `<rect x="${chartX}" y="${chartY - chartH}" width="${chartW}" height="${topDangerH}" fill="url(#stripes)" />`;
          chartHtml += `<line x1="${chartX}" y1="${dangerY}" x2="${chartX + chartW}" y2="${dangerY}" stroke="#000" stroke-dasharray="2,2" stroke-width="1.5"/>`; 
          chartHtml += `<text x="${chartX + chartW + 4}" y="${dangerY + 3}" font-size="10" class="t">5</text>`;
      }

      let pathD = "";
      let stepX = chartW / Math.max(1, aurHist.length - 1);
      
      aurHist.forEach((pt, i) => {
          let val = parseFloat(pt.n_ai);
          if (isNaN(val)) val = 0;
          let h = (val / maxAi) * chartH;
          let x = chartX + i * stepX;
          let y = chartY - h;
          
          if (i === 0) pathD += `M ${x.toFixed(1)},${y.toFixed(1)} `;
          else pathD += `L ${x.toFixed(1)},${y.toFixed(1)} `;
      });

      chartHtml += `<path d="${pathD}" fill="none" stroke="#000" stroke-width="3" stroke-linejoin="round" />`;
      chartHtml += `<line x1="${chartX}" y1="${chartY}" x2="${chartX + chartW}" y2="${chartY}" stroke="#000" stroke-width="2"/>`; 
      chartHtml += `<text x="${chartX + chartW + 4}" y="${chartY - chartH + 4}" font-size="10" class="t">${maxAi}</text>`;
  } else {
      chartHtml = `<text x="130" y="180" font-size="12" class="s" fill="#666" text-anchor="middle">Нет данных для графика</text>`;
  }

  let svg = `
  <svg xmlns="http://www.w3.org/2000/svg" width="260" height="240" viewBox="0 0 260 240">
    <defs>
      <pattern id="stripes" width="4" height="4" patternTransform="rotate(45)" patternUnits="userSpaceOnUse">
        <line x1="0" y1="0" x2="0" y2="4" stroke="#999" stroke-width="1.5" />
      </pattern>
    </defs>
    <style>
      .t { font-family: Arial, sans-serif; fill: #000; font-weight: bold; } 
      .s { font-family: Arial, sans-serif; fill: #000; font-weight: bold; }
      .l { font-family: Arial, sans-serif; fill: #000; font-size: 10px; }
    </style>
    <text x="10" y="20" font-size="16" class="t" letter-spacing="1">ПОЛЯРНЫЕ СИЯНИЯ</text>
    <line x1="10" y1="28" x2="250" y2="28" stroke="#000" stroke-width="3" stroke-linecap="round"/>

    <text x="10" y="55" font-size="11" class="s">ВЕРОЯТНОСТЬ</text>
    <text x="10" y="100" font-size="46" class="t">${auroraProb}</text>
    <text x="70" y="100" font-size="22" class="t">%</text>
    <rect x="10" y="115" width="114" height="16" rx="4" fill="none" stroke="#000" stroke-width="2"/>
    <rect x="12" y="117" width="${pBarWidth}" height="12" rx="2" fill="#000"/>
    <text x="10" y="142" class="l">0</text>
    <text x="124" y="142" class="l" text-anchor="end">100</text>
    
    <text x="135" y="55" font-size="11" class="s">ИНДЕКС (АИ)</text>
    <text x="135" y="100" font-size="46" class="t">${auroraIndex}</text>
    <rect x="135" y="115" width="114" height="16" rx="4" fill="none" stroke="#000" stroke-width="2"/>
    <rect x="137" y="117" width="${iBarWidth}" height="12" rx="2" fill="#000"/>
    <text x="135" y="142" class="l">0</text>
    <text x="249" y="142" class="l" text-anchor="end">10</text>

    ${chartHtml}
    <text x="130" y="232" font-size="14" class="t" text-anchor="middle">${statusText}</text>
  </svg>`;
  
  return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(svg.replace(/\n\s*/g, ''));
} catch (err) { return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(`<svg width="260" height="240"><text y="20">Ошибка: ${err.message}</text></svg>`); }

```

---

### 📌 Виджет 4: Солнечные вспышки

Выводит текущий индекс вспышечной активности и таблицу последних 5 вспышек с указанием класса, времени и области.

```javascript
try {
  let src = typeof $ === 'string' ? JSON.parse($) : $;
  let attr = src.attributes || {};

  let fIndex = (attr.flare_index !== undefined && attr.flare_index !== null && attr.flare_index !== "null") 
                ? Number(attr.flare_index).toFixed(1) 
                : "—";

  let statusText = attr.flare_status || src.state || "—";
  let flareSummary = attr.flare_summary || "Нет данных";

  let flares = [];
  try {
      let raw = attr.flares_list;
      if (typeof raw === 'string') raw = JSON.parse(raw);
      if (typeof raw === 'string') raw = JSON.parse(raw); 
      if (Array.isArray(raw)) flares = raw;
  } catch(e) {}
  
  while(flares.length < 5) flares.push({cls: "—", time: "—", reg: "—"});
  flares = flares.slice(0, 5);

  let rowsHtml = "";
  flares.forEach((f, i) => {
      let y = 125 + i * 22; 
      rowsHtml += `
        <text x="45" y="${y}" font-size="14" class="t" text-anchor="middle">${f.cls}</text>
        <text x="145" y="${y}" font-size="14" class="s" text-anchor="middle">${f.time}</text>
        <text x="225" y="${y}" font-size="14" class="t" text-anchor="middle">${f.reg}</text>
      `;
  });

  let svg = `
  <svg xmlns="http://www.w3.org/2000/svg" width="260" height="230" viewBox="0 0 260 230">
    <style>.t { font-family: Arial, sans-serif; fill: #000; font-weight: bold; } .s { font-family: Arial, sans-serif; fill: #000; font-weight: bold; }</style>
    
    <text x="10" y="20" font-size="16" class="t" letter-spacing="1">СОЛНЕЧНЫЕ ВСПЫШКИ</text>
    <line x1="10" y1="28" x2="250" y2="28" stroke="#000" stroke-width="3" stroke-linecap="round"/>

    <rect x="10" y="38" width="46" height="30" rx="6" fill="#000"/>
    <text x="33" y="58" font-size="16" fill="#fff" font-weight="bold" font-family="Arial" text-anchor="middle">${fIndex}</text>

    <text x="68" y="58" font-size="16" class="t">${statusText}</text>
    <text x="10" y="85" font-size="11" class="s">${flareSummary}</text>

    <text x="45" y="105" font-size="11" class="s" text-anchor="middle">класс</text>
    <text x="145" y="105" font-size="11" class="s" text-anchor="middle">время, МСК</text>
    <text x="225" y="105" font-size="11" class="s" text-anchor="middle">область</text>

    ${rowsHtml}
  </svg>`;
  
  return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(svg.replace(/\n\s*/g, ''));
} catch (err) { return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(`<svg width="260" height="230"><text y="20">Ошибка: ${err.message}</text></svg>`); }

```

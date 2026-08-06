# Погодный информер для Inker (Home Assistant + Yandex Pogoda)

<img width="1511" height="826" alt="Снимок экрана — 2026-08-06 в 11 46 50" src="https://github.com/user-attachments/assets/fdaab4b7-3c7b-48ec-b234-8bfda30131e9" />

Краткое руководство по настройке погодного информера для дисплеев Inker с использованием интеграции Yandex Pogoda из Home Assistant (HA).

У вас должна быть настроена интеграция Yandex Pogoda. Ниже приведена процедура создания вспомогательного сенсора.

### Настройка Home Assistant (Создание сенсора)

Вспомогательный сенсор собирает необходимые атрибуты для вывода на экран. Добавьте следующий код в конфигурацию HA (в раздел `packages` или `template`): `add_yandex_pogoda_sunset_sunrise.yaml`

```yaml
template:
  - trigger:
      # Запрашиваем стабильно раз в 15 минут, игнорируя промежуточные скачки статуса
      - platform: time_pattern
        minutes: "/15"
      # И при старте системы (оставляем)
      - platform: homeassistant
        event: start
    action:
      - service: weather.get_forecasts
        data:
          type: hourly
        target:
          entity_id: weather.yandex_pogoda
        response_variable: hourly_forecast
    sensor:
      - name: "Eink Weather Widget"
        unique_id: eink_weather_widget
        icon: mdi:weather-cloudy-alert
        state: "{{ states('weather.yandex_pogoda') }}"
        attributes:
          friendly_name: Eink Weather Widget
          temperature: "{{ state_attr('weather.yandex_pogoda', 'temperature') }}"
          apparent_temperature: "{{ state_attr('weather.yandex_pogoda', 'apparent_temperature') }}"
          yandex_condition: "{{ state_attr('weather.yandex_pogoda', 'yandex_condition') }}"
          # Оставляем всё как было у вас - это правильно!
          forecast_hourly: "{{ hourly_forecast['weather.yandex_pogoda'].forecast | to_json }}"
          next_rising: "{{ state_attr('sun.sun', 'next_rising') }}"
          next_setting: "{{ state_attr('sun.sun', 'next_setting') }}"
```

> **Проверка:** Перед установкой виджетов зайдите в HA в меню *Панель разработчика > Состояния* и убедитесь, что сущности `weather.yandex_pogoda` и `sensor.eink_weather_widget` существуют и отдают данные.


## 1. Способы установки

Установить информер можно двумя способами:

### Способ А: Быстрый импорт (Screen Code)
1. В Inker перейдите в раздел **Screens > Import**.
2. Откройте файл [yp_inker_screen.json](yp_inker_screen.json) в этом репозитории, скопируйте его содержимое целиком и нажмите **Import Screen**.
3. После импорта обязательно зайдите в раздел Data Sources, внесите изменения в:

**Источник 1 (Основной, Yandex Pogoda):** удалите и добавьте свои данные
*   **Basic Information**
    *   **Name:** `HA Yandex Pogoda`
*   **Connection**
    *   **URL:** `http://homeassistant.local:8123/api/states/weather.yandex_pogoda` проверьте ссылку
*   **Custom Headers** *(Add headers for authentication)*
    *   **Header name:** `Authorization`
    *   **Header value:** `Bearer ВАШ_ТОКЕН_HA`

**Источник 2 (Вспомогательный сенсор виджета):**
*   **Basic Information**
    *   **Name:** `HA Yandex Pogoda E-Paper`
    *   **Type:** `JSON API`
*   **Connection**
    *   **URL:** `http://homeassistant.local:8123/api/states/sensor.eink_weather_widget` проверьте ссылку
    *   **HTTP Method:** `GET`
*   **Custom Headers** *(Add headers for authentication)* удалите и добавьте свои данные
    *   **Header name:** `Authorization`
    *   **Header value:** `Bearer ВАШ_ТОКЕН_HA` 

--- 

### Способ Б: Ручная настройка
Если вы хотите собрать экран с нуля, сначала необходимо добавить два источника данных в разделе **Data Sources**, а затем вручную расставить виджеты (см. пункт 4).


## 2. Источники данных (Data Sources)

Для ручной настройки (Способ Б) создайте два источника со следующими параметрами:

**Источник 1 (Основной, Yandex Pogoda):**
*   **Basic Information**
    *   **Name:** `HA Yandex Pogoda`
    *   **Type:** `JSON API`
*   **Connection**
    *   **URL:** `http://homeassistant.local:8123/api/states/weather.yandex_pogoda`
    *   **HTTP Method:** `GET`
*   **Custom Headers** *(Add headers for authentication)*
    *   **Header name:** `Authorization`
    *   **Header value:** `Bearer ВАШ_ТОКЕН_HA`

**Источник 2 (Вспомогательный сенсор виджета):**
*   **Basic Information**
    *   **Name:** `HA Yandex Pogoda E-Paper`
    *   **Type:** `JSON API`
*   **Connection**
    *   **URL:** `http://homeassistant.local:8123/api/states/sensor.eink_weather_widget`
    *   **HTTP Method:** `GET`
*   **Custom Headers** *(Add headers for authentication)*
    *   **Header name:** `Authorization`
    *   **Header value:** `Bearer ВАШ_ТОКЕН_HA`

---

## 3. Ручное добавление виджетов (Библиотека)

Для каждого виджета из списка ниже необходимо применять **общие базовые настройки** (если не указано иное):
*   **Data Source:** `HA Yandex Pogoda` *(или `HA Yandex Pogoda` — см. указания к виджету)*
*   **Choose Display Type:** `Grid`
*   **Field:** `не влияет на наши виджеты, но к заполнению обязательно, ниже указаны для каждого виджета`
*   **Display As:** `Image` *(Обязательно для корректного отображения SVG-графики!)*
*   **Grid Settings:** Columns: `1`, Rows: `1`

Для каждого виджета перейдите в настройки ячейки (**Cell 0, 0**) и вставьте соответствующий код в блок **JavaScript Code**.

---

### 📌 Виджет 1: ЯП Направление и скорость ветра компас
*   **Data Source:** `HA Yandex Pogoda`
*   **Field:** `attributes.wind_bearing`

<details>
<summary><b>Показать код виджета (JavaScript)</b></summary>

```javascript
// 1. Берем градусы и скорость
let degrees = Math.round(parseFloat($.attributes.wind_bearing) || 0);

// Берем скорость из сенсора [110] (он сразу в м/с) или конвертируем из [230] (км/ч)
let speedRaw = parseFloat($.state);
if (isNaN(speedRaw)) {
    speedRaw = (parseFloat($.attributes.wind_speed) || 0) / 3.6;
}
let speed = speedRaw.toFixed(1);

// 2. Умный перевод градусов в буквы (С, СВ, В, ЮВ, Ю, ЮЗ, З, СЗ)
const dirs = ['С', 'СВ', 'В', 'ЮВ', 'Ю', 'ЮЗ', 'З', 'СЗ'];
let dirStr = dirs[Math.round(degrees / 45) % 8];

// 3. Генерируем тонкие и аккуратные риски
let ticks = "";
for (let i = 0; i < 360; i += 5) {
  if (i % 90 === 0) continue; 
  
  let isMedium = (i % 45 === 0);
  let length = isMedium ? 6 : 3;
  let weight = isMedium ? 2 : 1;
  
  ticks += `<line x1="50" y1="5" x2="50" y2="${5 + length}" stroke="#000" stroke-width="${weight}" transform="rotate(${i} 50 50)" />`;
}

// 4. Собираем SVG (Размер 180х180, внутри масштаб 100х100)
let svg = `
<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="180" height="180" viewBox="0 0 100 100">
  <!-- Строгое внешнее кольцо -->
  <circle cx="50" cy="50" r="47.5" fill="none" stroke="#000" stroke-width="5"/>
  
  <!-- Риски -->
  ${ticks}
  
  <!-- Стороны света (Кириллица) -->
  <text x="50" y="14" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" font-weight="900" fill="#000">С</text>
  <text x="89" y="53.5" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" font-weight="900" fill="#000">В</text>
  <text x="50" y="93" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" font-weight="900" fill="#000">Ю</text>
  <text x="11" y="53.5" text-anchor="middle" font-family="Arial, sans-serif" font-size="9" font-weight="900" fill="#000">З</text>
  
  <!-- ДАННЫЕ О ВЕТРЕ В ЦЕНТРЕ -->
  <text x="50" y="40" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" font-weight="bold" fill="#000">${speed} м/с</text>
  <text x="50" y="54" text-anchor="middle" font-family="Arial, sans-serif" font-size="15" font-weight="900" fill="#000">${dirStr}</text>
  <text x="50" y="67" text-anchor="middle" font-family="Arial, sans-serif" font-size="10" font-weight="bold" fill="#000">${degrees}°</text>

  <!-- КОНТУРНЫЙ ТРЕУГОЛЬНИК (прозрачный внутри) -->
  <g transform="rotate(${degrees} 50 50)">
    <polygon points="43,5 57,5 50,22" fill="none" stroke="#000" stroke-width="2.5" />
  </g>
</svg>
`;

return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(svg.replace(/\n\s*/g, ''));
```
</details>

---

### 📌 Виджет 2: ЯП График температуры на 12 часов
*(Примечание: скрипт автоматически обрабатывает прогноз на 24 часа для плавной визуализации)*
*   **Data Source:** `HA Yandex Pogoda`
*   **Field:** `attributes.forecast_hourly`

<details>
<summary><b>Показать код виджета (JavaScript)</b></summary>

```javascript
// 1. Берем РЕАЛЬНЫЕ данные
let forecast = $.attributes.forecast_hourly; 

if (!forecast || !Array.isArray(forecast) || forecast.length < 24) {
  return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(`<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="1000" height="180"><text x="10" y="30" fill="red">ОШИБКА: Нет данных на 24 часа</text></svg>`);
}

// 2. БЕРЕМ 24 ЧАСА
let data = forecast.slice(0, 24);
let temps = data.map(d => d.native_temperature);
let minT = Math.min(...temps);
let maxT = Math.max(...temps);
let range = maxT - minT || 1;

// 3. Настройки размеров
let w = 1200; 
let h = 180; 
let padTop = 60;
let padBot = 25; 
let padX = 25;   
let graphW = w - padX * 2;
let graphH = h - padTop - padBot;

// Библиотека иконок
let defs = `
  <defs>
    <g id="i-sun"><circle cx="12" cy="12" r="4"/><path d="M12 2v2"/><path d="M12 20v2"/><path d="m4.93 4.93 1.41 1.41"/><path d="m19.07 19.07 1.41 1.41"/><path d="M2 12h2"/><path d="M20 12h2"/><path d="m4.93 19.07 1.41-1.41"/><path d="m19.07 4.93-1.41 1.41"/></g>
    <g id="i-moon"><path d="M12 3a6 6 0 0 0 9 9 9 9 0 1 1-9-9Z"/></g>
    <g id="i-psun"><path d="M12 2v2"/><path d="m4.93 4.93 1.41 1.41"/><path d="M20 12h2"/><path d="m19.07 4.93-1.41 1.41"/><path d="M15.947 12.65a4 4 0 0 0-5.925-4.128"/><path d="M13 22H7a5 5 0 1 1 4.9-6H13a3 3 0 0 1 0 6Z"/></g>
    <g id="i-pmoon"><path d="M13 16a3 3 0 1 1 0 6H7a5 5 0 1 1 4.9-6Z"/><path d="M10.1 9A6 6 0 0 1 16 4a4.24 4.24 0 0 0 6 6 6 6 0 0 1-3 5.197"/></g>
    <g id="i-cloud"><path d="M17.5 19H9a7 7 0 1 1 6.71-9h1.79a4.5 4.5 0 1 1 0 9Z"/></g>
    <g id="i-rain"><path d="M4 14.899A7 7 0 1 1 15.71 8h1.79a4.5 4.5 0 0 1 2.5 8.242"/><path d="M16 14v6"/><path d="M8 14v6"/><path d="M12 16v6"/></g>
    <g id="i-snow"><path d="M4 14.899A7 7 0 1 1 15.71 8h1.79a4.5 4.5 0 0 1 2.5 8.242"/><path d="M8 15h.01"/><path d="M8 19h.01"/><path d="M12 17h.01"/><path d="M12 21h.01"/><path d="M16 15h.01"/><path d="M16 19h.01"/></g>
  </defs>
`;

let points = "";
let elements = "";

// 4. Высчитываем точки
data.forEach((item, i) => {
  let temp = Math.round(item.native_temperature);
  let date = new Date(item.datetime);
  let hours = date.getHours();
  let hoursStr = hours.toString().padStart(2, '0');
  
  let cond = item.condition || "cloudy";
  let isNight = (hours >= 21 || hours < 5); 
  let iconId = "i-cloud"; 
  
  if (cond === 'clear-night') iconId = 'i-moon';
  else if (cond === 'sunny' || cond === 'clear') iconId = isNight ? 'i-moon' : 'i-sun';
  else if (cond === 'partlycloudy') iconId = isNight ? 'i-pmoon' : 'i-psun';
  else if (cond.includes('rain') || cond === 'pouring' || cond === 'hail') iconId = 'i-rain';
  else if (cond.includes('snow')) iconId = 'i-snow';

  let x = padX + (graphW / (data.length - 1)) * i;
  let y = padTop + graphH - ((temp - minT) / range) * graphH;

  points += `${x},${y} `;
  
  elements += `<use href="#${iconId}" x="${x - 12}" y="${y - 48}" width="24" height="24" stroke="#000" stroke-width="2" fill="none" stroke-linecap="round" stroke-linejoin="round"/>`;
  elements += `<text x="${x}" y="${y - 12}" text-anchor="middle" font-family="Arial, sans-serif" font-size="16" font-weight="900" fill="#000">${temp}°</text>`;
  elements += `<text x="${x}" y="${h - 5}" text-anchor="middle" font-family="Arial, sans-serif" font-size="14" fill="#000">${hoursStr}:00</text>`;
  elements += `<circle cx="${x}" cy="${y}" r="4" fill="#fff" stroke="#000" stroke-width="2.5"/>`;
});

let svg = `
<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="${w}" height="${h}" viewBox="0 0 ${w}${h}">
  ${defs}
  <polyline points="${points.trim()}" fill="none" stroke="#000" stroke-width="3" stroke-linejoin="round"/>
  ${elements}
</svg>`;

return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(svg.replace(/\n\s*/g, ''));
```
</details>

---

### 📌 Виджет 3: ЯП График сила и направление ветра на 12 ч
*   **Data Source:** `HA Yandex Pogoda`
*   **Field:** `attributes.forecast_hourly`

<details>
<summary><b>Показать код виджета (JavaScript)</b></summary>

```javascript
// 1. Берем РЕАЛЬНЫЕ данные
let forecast = $.attributes.forecast_hourly; 

if (!forecast || !Array.isArray(forecast) || forecast.length < 12) {
  return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(`<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="600" height="180"><text x="10" y="30" fill="red">ОШИБКА данных ветра</text></svg>`);
}

let data = forecast.slice(0, 12);
let speeds = data.map(d => Number(d.native_wind_speed) || 0);
let minS = Math.min(...speeds);
let maxS = Math.max(...speeds);
let range = maxS - minS || 1; 

let w = 600; 
let h = 180; 
let padTop = 75; 
let padBot = 25; 
let padX = 25;   
let graphW = w - padX * 2;
let graphH = h - padTop - padBot;

function getCardinal(angle) {
  const dirs = ['С', 'СВ', 'В', 'ЮВ', 'Ю', 'ЮЗ', 'З', 'СЗ'];
  return dirs[Math.round(angle / 45) % 8];
}

let defs = `
  <defs>
    <g id="wind-arrow">
      <line x1="0" y1="7" x2="0" y2="-7" stroke="#000" stroke-width="2" stroke-linecap="round"/>
      <polyline points="-4,-3 0,-8 4,-3" fill="none" stroke="#000" stroke-width="2" stroke-linejoin="round" stroke-linecap="round"/>
    </g>
  </defs>
`;

let points = "";
let elements = "";

data.forEach((item, i) => {
  let speed = Number(item.native_wind_speed || 0).toFixed(1); 
  let bearing = item.wind_bearing || 0;
  let dirStr = getCardinal(bearing);
  let date = new Date(item.datetime);
  let hoursStr = date.getHours().toString().padStart(2, '0');
  
  let x = padX + (graphW / 11) * i;
  let y = padTop + graphH - ((speed - minS) / range) * graphH;

  points += `${x},${y} `;
  elements += `<use href="#wind-arrow" x="${x}" y="${y - 58}" transform="rotate(${bearing + 180} ${x}${y - 58})" />`;
  elements += `<text x="${x}" y="${y - 35}" text-anchor="middle" font-family="Arial, sans-serif" font-size="12" font-weight="bold" fill="#000">${dirStr}</text>`;
  elements += `<text x="${x}" y="${y - 15}" text-anchor="middle" font-family="Arial, sans-serif" font-size="15" font-weight="900" fill="#000">${speed}</text>`;
  elements += `<text x="${x}" y="${h - 5}" text-anchor="middle" font-family="Arial, sans-serif" font-size="14" fill="#000">${hoursStr}:00</text>`;
  elements += `<circle cx="${x}" cy="${y}" r="4" fill="#fff" stroke="#000" stroke-width="2.5"/>`;
});

let svg = `
<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="${w}" height="${h}" viewBox="0 0 ${w}${h}">
  ${defs}
  <polyline points="${points.trim()}" fill="none" stroke="#000" stroke-width="3" stroke-linejoin="round"/>
  ${elements}
</svg>`;

return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(svg.replace(/\n\s*/g, ''));
```
</details>

---

### 📌 Виджет 4: ЯП Прогноз на завтра
*   **Data Source:** `HA Yandex Pogoda`
*   **Field:** `entity_id`

<details>
<summary><b>Показать код виджета (JavaScript)</b></summary>

```javascript
// 1. Берем РЕАЛЬНЫЕ данные
let forecast = $?.attributes?.forecast_hourly; 

if (!forecast || !Array.isArray(forecast) || forecast.length === 0) {
  let errSvg = `<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="800" height="240"><text x="10" y="40" font-family="Arial, sans-serif" font-size="26" fill="red">ОШИБКА: Прогноз не найден!</text></svg>`;
  return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(errSvg.replace(/\n\s*/g, ''));
}

// 2. ЖЕЛЕЗНАЯ ЛОГИКА КАЛЕНДАРНОГО ЗАВТРА
let todayStr = forecast[0].datetime.split('T')[0];
let futureData = forecast.filter(d => !d.datetime.startsWith(todayStr));

let tomorrowData = futureData;
if (futureData.length > 0) {
  let tomorrowStr = futureData[0].datetime.split('T')[0];
  tomorrowData = futureData.filter(d => d.datetime.startsWith(tomorrowStr));
} else {
  tomorrowData = forecast.slice(-12);
}

// 3. Вытаскиваем значения строго для ЗАВТРАШНЕГО дня
let temps = tomorrowData.map(d => d.native_temperature);
let feels = tomorrowData.map(d => d.native_apparent_temperature ?? d.native_temperature);
let wSpeeds = tomorrowData.map(d => Number(d.native_wind_speed) || 0);
let wGusts = tomorrowData.map(d => Number(d.native_wind_gust_speed) || 0);

let maxTemp = Math.max(...temps);
let maxFeels = Math.max(...feels);
let minWind = Math.min(...wSpeeds);
let maxWind = Math.max(...wSpeeds);
let maxGust = Math.max(...wGusts);

let midDayIndex = Math.floor(tomorrowData.length / 2);
let bearing = tomorrowData[midDayIndex].wind_bearing || 0; 

function getCardinal(angle) {
  const dirs = ['С', 'СВ', 'В', 'ЮВ', 'Ю', 'ЮЗ', 'З', 'СЗ'];
  return dirs[Math.round(angle / 45) % 8];
}

// 4. Ищем часы с осадками на ЗАВТРА
let rainHours = tomorrowData
  .filter(d => d.condition && (d.condition.includes('rain') || d.condition === 'snowy' || d.condition === 'pouring' || d.condition === 'hail'))
  .map(d => new Date(d.datetime).getHours());

let rainText = "Осадков не ожидается";
if (rainHours.length > 0) {
  let groups = [];
  let start = rainHours[0], prev = rainHours[0];
  
  for (let i = 1; i < rainHours.length; i++) {
    if (rainHours[i] === prev + 1) {
      prev = rainHours[i];
    } else {
      groups.push(`с ${start}:00 до ${prev + 1}:00`);
      start = rainHours[i]; prev = rainHours[i];
    }
  }
  let endHour = prev + 1 === 24 ? '24' : prev + 1;
  groups.push(`с ${start}:00 до ${endHour}:00`);
  rainText = "Местами осадки " + groups.join(' и ');
}

// 5. Формируем строки с заголовком "Завтра"
let tSign = maxTemp > 0 ? "+" : "";
let fSign = maxFeels > 0 ? "+" : "";

let textTemp = `Завтра: до ${tSign}${Math.round(maxTemp)}° (ощущается как ${fSign}${Math.round(maxFeels)}°).`;
let textRain = rainText + ".";
let textWind = `Ветер ${getCardinal(bearing)}, от ${Math.round(minWind)} до ${Math.round(maxWind)} м/с, порывы до ${Math.round(maxGust)} м/с.`;

// 6. Собираем SVG-карточку
let w = 800; 
let h = 240; 

let svg = `
<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="${w}" height="${h}" viewBox="0 0 ${w}${h}">
  <defs>
    <g id="i-sun"><path d="M12 2v2"/><path d="m4.93 4.93 1.41 1.41"/><path d="M20 12h2"/><path d="m19.07 4.93-1.41 1.41"/><path d="M15.947 12.65a4 4 0 0 0-5.925-4.128"/><path d="M13 22H7a5 5 0 1 1 4.9-6H13a3 3 0 0 1 0 6Z"/></g>
    <g id="i-umbrella"><path d="M12 2v20"/><path d="M20.5 10A8.5 8.5 0 0 0 3.5 10h17Z"/><path d="M12 22a2 2 0 0 0 2-2"/></g>
    <g id="i-wind"><path d="M4 12h12a2 2 0 0 0 0-4h-2"/><path d="M4 16h8a2 2 0 0 1 0 4h-2"/><path d="M4 8h6a2 2 0 0 1 0-4h-2"/></g>
  </defs>
  <style>
    .t { font-family: Arial, sans-serif; font-size: 26px; fill: #000; font-weight: bold; }
    .i { stroke: #000; stroke-width: 3; fill: none; stroke-linecap: round; stroke-linejoin: round; }
  </style>
  <use href="#i-sun" class="i" x="10" y="15" width="46" height="46"/>
  <text x="75" y="47" class="t">${textTemp}</text>
  <use href="#i-umbrella" class="i" x="10" y="90" width="46" height="46"/>
  <text x="75" y="122" class="t">${textRain}</text>
  <use href="#i-wind" class="i" x="10" y="165" width="46" height="46"/>
  <text x="75" y="197" class="t">${textWind}</text>
</svg>
`;

return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(svg.replace(/\n\s*/g, ''));
```
</details>

---

### 📌 Виджет 5: ЯП Прогноз на два дня краткий
*   **Data Source:** `HA Yandex Pogoda`
*   **Field:** `entity_id`

<details>
<summary><b>Показать код виджета (JavaScript)</b></summary>

```javascript
// 1. ПРОГНОЗ НА 2 ДНЯ (Берем только дневные)
let forecastDaily = $?.attributes?.forecast_twice_daily || [];
let days = forecastDaily.filter(d => d.is_daytime === true).slice(0, 2);

if (days.length === 0) {
  return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(`<svg width="270" height="180"><text y="20" fill="red">Нет данных</text></svg>`);
}

let w = 270; 
let h = 180;
let elements = "";
const daysOfWeek = ['ВС', 'ПН', 'ВТ', 'СР', 'ЧТ', 'ПТ', 'СБ'];

function getIconId(cond) {
  if (!cond) return "i-cloud";
  if (cond.includes('clear')) return 'i-sun';
  if (cond.includes('partly')) return 'i-psun';
  if (cond.includes('rain') || cond.includes('pour') || cond.includes('shower') || cond.includes('hail')) return 'i-rain';
  if (cond.includes('snow') || cond.includes('sleet')) return 'i-snow';
  return "i-cloud";
}

let defs = `
  <defs>
    <g id="i-sun"><circle cx="12" cy="12" r="5"/><path d="M19 12h3 M12 19v3 M5 12H2 M12 5V2 M16.95 16.95l2.12 2.12 M7.05 16.95l-2.12 2.12 M7.05 7.05l-2.12-2.12 M16.95 7.05l2.12-2.12"/></g>
    <g id="i-psun"><path d="M19 12h3 M12 5V2 M16.95 16.95l2.12 2.12 M7.05 7.05l-2.12-2.12 M16.95 7.05l2.12-2.12"/><path d="M16 13a4 4 0 0 0-6-4.13"/><path d="M13 22H7a5 5 0 1 1 4.9-6H13a3 3 0 0 1 0 6Z"/></g>
    <g id="i-cloud"><path d="M17.5 19H9a7 7 0 1 1 6.7-9h1.8a4.5 4.5 0 1 1 0 9Z"/></g>
    <g id="i-rain"><path d="M4 15A7 7 0 1 1 15.7 8h1.8a4.5 4.5 0 0 1 2.5 8.2"/><path d="M16 14v6m-8-6v6m4-4v6"/></g>
    <g id="i-snow"><path d="M4 15A7 7 0 1 1 15.7 8h1.8a4.5 4.5 0 0 1 2.5 8.2"/><path d="M8 15h.01m0 4h.01m4-2h.01m0 4h.01m4-6h.01m0 4h.01"/></g>
  </defs>
`;

days.forEach((day, i) => {
  let date = new Date(day.datetime);
  let name = daysOfWeek[date.getDay()] + " " + date.getDate().toString().padStart(2,'0') + "." + (date.getMonth()+1).toString().padStart(2,'0');
  let high = Math.round(day.native_temperature);
  let low = Math.round(day.native_templow || (high-5));
  let icon = getIconId(day.condition);
  
  let hasRain = day.condition && (day.condition.includes('rain') || day.condition === 'snowy' || day.condition === 'pouring' || day.condition === 'hail' || day.condition === 'showers');
  let rainText = hasRain ? "Осадки" : "Без осадков";
  
  let x = days.length === 1 ? (w/2) : (i === 0 ? 65 : 205); 
  
  elements += `
    <text x="${x}" y="25" text-anchor="middle" font-family="Arial, sans-serif" font-size="20" font-weight="bold" fill="#000">${name}</text>
    <text x="${x}" y="65" text-anchor="middle" font-family="Arial, sans-serif" font-size="28" font-weight="bold" fill="#000">${high}°</text>
    <g transform="translate(${x-20}, 78) scale(1.6)"><use href="#${icon}" stroke="#000" stroke-width="2" fill="none" stroke-linecap="round" stroke-linejoin="round"/></g>
    <text x="${x}" y="145" text-anchor="middle" font-family="Arial, sans-serif" font-size="22" font-weight="bold" fill="#000">~${low}°</text>
    <text x="${x}" y="170" text-anchor="middle" font-family="Arial, sans-serif" font-size="15" font-weight="bold" fill="#000">${rainText}</text>
  `;
});

let svg = `
<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="${w}" height="${h}" viewBox="0 0 ${w}${h}">
  ${defs}${elements}
</svg>`;

return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(svg.replace(/\n\s*/g, ''));
```
</details>

---

### 📌 Виджет 6: ЯП Сегодня основной
> **Внимание:** Для этого виджета необходимо переключить **Data Source** на вспомагательный источник интеграции!
*   **Data Source:** `HA Yandex Pogoda E-Paper`
*   **Field:** `entity_id`

<details>
<summary><b>Показать код виджета (JavaScript)</b></summary>

```javascript
try {
  // 1. БЕРЕМ ВСЕ РЕАЛЬНЫЕ ДАННЫЕ ИЗ НАШЕГО СБОРНОГО ДАТЧИКА
  let src = typeof $ === 'string' ? JSON.parse($) : $;
  let weather = src.attributes || {};
  
  // 1.1 ПРАВИЛЬНАЯ РАСПАКОВКА ПРОГНОЗА
  let forecast = [];
  try {
    if (typeof weather.forecast_hourly === 'string') {
      forecast = JSON.parse(weather.forecast_hourly);
    } else if (Array.isArray(weather.forecast_hourly)) {
      forecast = weather.forecast_hourly;
    }
  } catch(e) {}

  // 2. БАЗОВЫЕ ЗНАЧЕНИЯ
  let t_curr = Math.round(weather.temperature ?? 0);
  let t_feels = Math.round(weather.apparent_temperature ?? 0);

  // 3. РАССВЕТ И ЗАКАТ
  let formatTime = (iso) => {
    if(!iso) return "--:--";
    let d = new Date(iso);
    return d.getHours().toString().padStart(2,'0') + ":" + d.getMinutes().toString().padStart(2,'0');
  };
  let sunrise = formatTime(weather.next_rising);
  let sunset = formatTime(weather.next_setting);

  // 4. ТЕКСТ ПОГОДЫ И УМНЫЕ ИКОНКИ
  let cond_raw = weather.yandex_condition || src.state || 'clear';
  const condMap = {
    'clear': 'Ясно', 'clear-night': 'Ясно', 
    'partly_cloudy': 'Малооблачно', 'partlycloudy': 'Малооблачно',
    'cloudy': 'Облачно', 'overcast': 'Пасмурно',
    'light_rain': 'Неб. дождь', 'rain': 'Дождь', 'rainy': 'Дождь',
    'heavy_rain': 'Сильный дождь', 'showers': 'Ливень', 'pouring': 'Ливень',
    'sleet': 'Мокрый снег', 'light_snow': 'Неб. снег', 'snow': 'Снег', 'snowy': 'Снег', 'snowfall': 'Снегопад',
    'hail': 'Град', 'thunderstorm': 'Гроза', 'lightning': 'Гроза'
  };
  let condText = condMap[cond_raw] || 'Облачно';

  let isNight = (new Date().getHours() >= 21 || new Date().getHours() < 5); 
  let iconId = "i-cloud"; 
  if (cond_raw.includes('clear')) iconId = isNight ? 'i-moon' : 'i-sun';
  else if (cond_raw.includes('partly')) iconId = isNight ? 'i-pmoon' : 'i-psun';
  else if (cond_raw.match(/rain|pour|shower|hail|light/)) iconId = 'i-rain';
  else if (cond_raw.match(/snow|sleet/)) iconId = 'i-snow';

  // 5. АНАЛИЗАТОР СТРОГО НА СЕГОДНЯ
  let minTemp = t_curr;
  let maxTemp = t_curr;
  let rainText = "Сегодня без осадков.";

  if (forecast && forecast.length > 0) {
    // Берем дату самого первого часа (это и есть "сегодня")
    let todayStr = forecast[0].datetime.split('T')[0];
    
    // Оставляем часы СТРОГО до полуночи текущего дня
    let todayData = forecast.filter(d => d.datetime && d.datetime.startsWith(todayStr));
    
    // Фолбек на случай сбоя дат (берем ближайшие 12 часов)
    if (todayData.length === 0) todayData = forecast.slice(0, 12);

    let temps = todayData.map(d => d.native_temperature).filter(t => t !== undefined);
    if (temps.length > 0) {
      minTemp = Math.min(...temps);
      maxTemp = Math.max(...temps);
    }

    // Умный поиск ЛЮБЫХ видов осадков (ищет по корням слов)
    let rainHours = todayData
      .filter(d => d.condition && d.condition.match(/rain|snow|sleet|pour|shower|hail|thunder|lightning/i))
      .map(d => new Date(d.datetime).getHours());

    if (rainHours.length > 0) {
      let groups = [];
      let start = rainHours[0], prev = rainHours[0];
      
      for (let i = 1; i < rainHours.length; i++) {
        if (rainHours[i] === prev + 1) { 
            prev = rainHours[i]; 
        } else {
          groups.push(`с ${start}:00 до ${prev + 1}:00`);
          start = rainHours[i]; prev = rainHours[i];
        }
      }
      let endHour = prev + 1 === 24 ? '24' : prev + 1;
      groups.push(`с ${start}:00 до ${endHour}:00`);
      rainText = "Сегодня осадки " + groups.join(' и ') + ".";
    }
  }

  let fSign = t_feels > 0 ? "+" : "";
  let minSign = minTemp > 0 ? "+" : "";
  let maxSign = maxTemp > 0 ? "+" : "";

  // 6. СОБИРАЕМ ИДЕАЛЬНЫЙ SVG (строго 300x180)
  let w = 300;
  let h = 180;

  let svg = `
  <svg xmlns="http://www.w3.org/2000/svg" width="${w}" height="${h}" viewBox="0 0 600 360">
    <defs>
      <g id="i-sun">
        <circle cx="12" cy="12" r="5"/>
        <path d="M19 12h3 M12 19v3 M5 12H2 M12 5V2 M16.95 16.95l2.12 2.12 M7.05 16.95l-2.12 2.12 M7.05 7.05l-2.12-2.12 M16.95 7.05l2.12-2.12"/>
      </g>
      <g id="i-moon"><path d="M12 3a6 6 0 0 0 9 9 9 9 0 1 1-9-9Z"/></g>
      <g id="i-psun">
        <path d="M19 12h3 M12 5V2 M16.95 16.95l2.12 2.12 M7.05 7.05l-2.12-2.12 M16.95 7.05l2.12-2.12"/>
        <path d="M16 13a4 4 0 0 0-6-4.13"/>
        <path d="M13 22H7a5 5 0 1 1 4.9-6H13a3 3 0 0 1 0 6Z"/>
      </g>
      <g id="i-pmoon"><path d="M13 16a3 3 0 1 1 0 6H7a5 5 0 1 1 4.9-6Z"/><path d="M10.1 9A6 6 0 0 1 16 4a4.24 4.24 0 0 0 6 6 6 6 0 0 1-3 5.2"/></g>
      <g id="i-cloud"><path d="M17.5 19H9a7 7 0 1 1 6.7-9h1.8a4.5 4.5 0 1 1 0 9Z"/></g>
      <g id="i-rain"><path d="M4 15A7 7 0 1 1 15.7 8h1.8a4.5 4.5 0 0 1 2.5 8.2"/><path d="M16 14v6m-8-6v6m4-4v6"/></g>
      <g id="i-snow"><path d="M4 15A7 7 0 1 1 15.7 8h1.8a4.5 4.5 0 0 1 2.5 8.2"/><path d="M8 15h.01m0 4h.01m4-2h.01m0 4h.01m4-6h.01m0 4h.01"/></g>
      
      <g id="i-sr"><path d="M2 18h20 M7 18a5 5 0 0 1 10 0 M12 9v-4 M18 11l3-3 M6 11l-3-3 M22 15h-3 M5 15H2"/></g>
      
      <g id="i-ss">
        <path d="M2 18h20 M12 5v4 M18 11l3-3 M6 11l-3-3 M22 15h-3 M5 15H2"/>
        <path d="M7 18a5 5 0 0 1 10 0Z" fill="#000"/>
      </g>
      
      <g id="i-umb"><path d="M12 2v20 M20.5 10A8.5 8.5 0 0 0 3.5 10h17Z M12 22a2 2 0 0 0 2-2"/></g>
    </defs>
    <style>
      .t { font-family: Arial, sans-serif; fill: #000; font-weight: bold; }
      .i { stroke: #000; fill: none; stroke-linecap: round; stroke-linejoin: round; }
    </style>

    <text x="150" y="140" font-size="130" text-anchor="end" class="t">${t_curr}</text>
    <circle cx="170" cy="55" r="14" stroke="#000" stroke-width="7" fill="none" />

    <g transform="translate(210, 35) scale(1.6)"><use href="#i-sr" class="i" stroke-width="2.5"/></g>
    <text x="260" y="60" font-size="24" class="t">${sunrise}</text>
    
    <g transform="translate(210, 100) scale(1.6)"><use href="#i-ss" class="i" stroke-width="2.5"/></g>
    <text x="260" y="125" font-size="24" class="t">${sunset}</text>

    <g transform="translate(420, 15) scale(6)"><use href="#${iconId}" class="i" stroke-width="1.3"/></g>

    <text x="20" y="220" font-size="30" class="t">Ощущается: ${fSign}${t_feels}°</text>
    
    <path d="M25 245 v20 M18 258 L25 265 L32 258" class="i" stroke-width="3"/>
    <text x="50" y="263" font-size="26" class="t">${minSign}${minTemp}°</text>
    
    <path d="M140 265 v-20 M133 252 L140 245 L147 252" class="i" stroke-width="3"/>
    <text x="165" y="263" font-size="26" class="t">${maxSign}${maxTemp}°</text>

    <text x="560" y="263" font-size="34" text-anchor="end" class="t">${condText}</text>

    <line x1="20" y1="300" x2="580" y2="300" stroke="#ddd" stroke-width="2" stroke-linecap="round"/>

    <g transform="translate(20, 315) scale(1.3)"><use href="#i-umb" class="i" stroke-width="2.5"/></g>
    <text x="70" y="340" font-size="24" class="t">${rainText}</text>
  </svg>`;

  return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(svg.replace(/\n\s*/g, ''));
} catch (err) {
  return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(`<svg width="300" height="180"><text y="20" font-family="Arial" font-size="14" fill="#000">Ошибка: ${err.message}</text></svg>`);
}
```
</details>

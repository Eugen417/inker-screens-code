Погодный информер для Inker использующий в качестве источника интеграцию для Home Assistant, Yandex Pogoda.
Данные из HA в Inker передаются посредством двух источников(Data Source):
1) Непосредственно из интеграции Yandex Pogoda, Data Source: HA Yandex Pogoda E-Paper, Connection URL `http://homeassistant.loc:8123/api/states/weather.yandex_pogoda`
2) и вспомогательного датчика Eink Weather Widget, Data Source: HA Yandex Pogoda, Connection URL `http://homeassistant.loc:8123/api/states/sensor.eink_weather_widget`

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

Ручная добавлениеи настройка виджетов:
ЯП Направление и скорость ветра компас
Истчник(Data Source): `HA Yandex Pogoda`
Choose Display Type: `Grid`
Widget Name: `ЯП Направление и скорость ветра компас`
Grid Settings: Columns: `1`, Rows: `1`
Cell 0, 0
  Field: `attributes.wind_bearing`
  JavaScript Code:
  ```JavaScript
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
<svg xmlns="http://www.w3.org/2000/svg" width="180" height="180" viewBox="0 0 100 100">
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

// 5. Пакуем в строку и возвращаем картинку
return "data:image/svg+xml;charset=utf-8," + encodeURIComponent(svg.replace(/\n\s*/g, ''));
  ```

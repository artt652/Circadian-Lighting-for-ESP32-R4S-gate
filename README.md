# Circadian-Lighting-for-ESP32-R4S-gate

Яркость экрана которая комфортная для глаз днём, в большинстве случаев слишком высокая для вечера и ночи. Сначала хотел  сделать автоматизацию на изменение яркости по времени, потом подумал что сейчас темнеет все позже. Решил привязать автоматизацию к заходу / восхода солнца, а потом вспомнил что есть шикарная готовая интеграция Circadian Lighting, которая это все делает сама в автоматическом режиме. 

Однако оказалось что яркость шлюза регулируется в домене number, с которым эта интеграция не работает. 
 
Решение оказалось простым -  использовать стандартный компонент ХА - Template Light. 

В конфигурации нужно прописать следующий код, (в r4s5. надо заменить "5" на  номер своего шлюза):

 light:
  - platform: template
    lights:
      r4sxx_gate_screen:
        friendly_name: "Подсветка экрана"
        availability_template: "{{states('sensor.r4s5_gate_rssi') | int}}"
        turn_off:
          service: number.set_value
          data:
            value: '0'
            entity_id: number.r4s5_gate_screen
        turn_on:
          service: number.set_value
          data:
            value: '10' #яркость при включении
            entity_id: number.r4s5_gate_screen
        set_level:
          service: number.set_value
          data:
            value: "{{ brightness }}"
            entity_id: number.r4s5_gate_screen

+ добавить в кофиг интеграции Circadian Lighting (также для себя установил значение максимальной яркости в 20%) :

  - platform: circadian_lighting
    #max_brightness: 20 # (1-100)
    name: Автояркость экрана
    lights_brightness:
       - light.r4sxx_gate_screen

После перезагрузки ХА появятся 2 новых объекта: 

1. Новый источник света, это и есть подсветка шлюза, с регулировкой яркости; 
2. Выключатель адаптивной яркости, после его включения яркость экрана начинает подстравивается под время суток автоматически.

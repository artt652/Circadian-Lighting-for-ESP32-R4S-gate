# Circadian-Lighting-for-ESP32-R4S-gate

Яркость экрана которая комфортная для глаз днём, в большинстве случаев слишком высокая для вечера и ночи. 
Сначала хотел  сделать автоматизацию на изменение яркости по времени, или привязать автоматизацию к заходу / восхода солнца, а потом вспомнил что есть шикарная готовая интеграция [Circadian Lighting](https://github.com/claytonjn/hass-circadian_lighting), которая это все делает сама в автоматическом режиме. 

Однако оказалось что яркость шлюза регулируется в домене number, т.к на текущий момент это единственная возможность передать и принять число по mqtt, но с которым эта интеграция (пока?) не работает. 
 
Решение оказалось простым -  использовать стандартный компонент ХА - [Template Light](https://www.home-assistant.io/integrations/light.template/). Для этого в configuration.yaml нужно прописать следующий код, и в тексте r4s5_gate заменить "5" на  номер своего шлюза:
```yaml
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
            value: '10'
            entity_id: number.r4s5_gate_screen
        set_level:
          service: number.set_value
          data:
            value: "{{ brightness }}"
            entity_id: number.r4s5_gate_screen
```
+ добавить в конфигурацию запись для интеграции Circadian Lighting:
```yaml
switch:
  - platform: circadian_lighting
    #max_brightness: 20 # (1-100) значение максимальной яркости
    name: Автояркость экрана
    lights_brightness:
       - light.r4sxx_gate_screen
```
Кроме того, можно сюда же ещё добавить и выключатель для перезагрузки ESP32:

```yaml
  - platform: template
    switches:
      r4s_gate_restart: # В явном виде кнопки нет. 
                        # Но возможность рестарта по мктт есть.  
                        # Для этого в топик screen нужно
                        # записать restart, reset или reboot.
        friendly_name: Перезагрузка ESP32
        value_template: "{{ is_state('sensor.r4s5_gate_rssi', '0') }}"
        availability_template: "{{states('sensor.r4s5_gate_rssi') | int}}"
        turn_on:
          service: mqtt.publish
          data:
            topic: r4s5/screen
            payload: restart
        turn_off:
```
После перезагрузки ХА появятся 3 новых объекта: 

1. Новый источник света, **light.r4sxx_gate_screen**, это и есть подсветка шлюза, с регулировкой яркости; 
2. Выключатель адаптивной яркости, **switch.circadian_lighting_avtoiarkost_ekrana**, после его включения яркость экрана начинает подстравивается под время суток автоматически.
3. Выключатель для перезагрузки ESP32 **switch.r4s_gate_restart**.

Также можно настроить:
[Автообновляемый прогноз погоды на экране](https://github.com/artt652/Weather-for-ESP32-R4S-gate)

[Обсуждение в телеграм](https://t.me/ESP32_R4sGate)

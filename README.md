# ESPHome Windrose para M5Stack Fire

Uso de un M5Stack Fire para presentar datos meteorológicos de Home Assistant.

Diseño basado en la tarjeta de Home Assistant https://github.com/aukedejong/lovelace-windrose-card

## 1. Configura las entidades

En `m5-stack-fire-windrose.yaml`, ajusta las entidades de `substitutions`:

```yaml
substitutions:
  wind_direction_entity: sensor.ecowitt_wind_direction
  wind_speed_entity: sensor.ecowitt_wind_speed
  wind_gust_entity: sensor.ecowitt_wind_gust
  cloud_base_entity: sensor.ecowitt_cloud_base
  temperature_entity: sensor.ecowitt_outdoor_temperature
  humidity_entity: sensor.ecowitt_humidity
  apparent_temperature_entity: sensor.ecowitt_apparent_temperature
  solar_radiation_entity: sensor.ecowitt_solar_radiation
  uv_index_entity: sensor.ecowitt_uv_index
  solar_radiation_max_entity: sensor.ecowitt_max_solar_radiation
  rain_rate_entity: sensor.ecowitt_rain_rate
  rain_rate_hour_entity: sensor.ecowitt_rain_rate_hour
  rain_last_hour_entity: sensor.ecowitt_rain_last_hour
  rain_24h_entity: sensor.ecowitt_rain_24h
  rain_total_entity: sensor.ecowitt_rain_total
  barometer_entity: sensor.ecowitt_barometer
  sunrise_entity: sensor.sun_next_rising
  sunset_entity: sensor.sun_next_setting
```

Usa los `entity_id` reales de Home Assistant. La pantalla espera temperatura
en °C, humedad en %, radiación solar en W/m², techo de nubes en metros,
barómetro en mbar y lluvia en mm/h o mm.

## 2. Pantallas y botones

Los tres botones físicos seleccionan una pantalla y actualizan el display al
instante:

| Botón | Pantalla |
| --- | --- |
| A | Lluvia |
| B | Temperatura y humedad, con gráfica de las últimas 12 horas |
| C | Rosa de los vientos |

Las muestras de viento, temperatura y humedad se toman cada 17 segundos
mientras la API de Home Assistant está conectada. Los históricos muestran las
últimas 12 horas. Los botones dejan de ajustar el brillo para dedicarse por
completo a la navegación.

La retroiluminación se atenúa automáticamente al 25 % entre las 23:30 y las
07:30, según la hora sincronizada desde Home Assistant; fuera de ese intervalo
permanece al 100 %.

## 3. Empareja el dispositivo con Home Assistant

Los sensores `platform: homeassistant` solo reciben valores cuando Home Assistant esta conectado al dispositivo por la Native API de ESPHome.

Si usas cifrado, el bloque debe quedar asi:

```yaml
api:
  encryption:
    key: !secret m5_stack_fire_encryption_key
```

Y en `secrets.yaml` debe existir esa clave:

```yaml
m5_stack_fire_encryption_key: "TU_CLAVE_GENERADA"
```

Despues de compilar y subir el firmware:

1. Ve a Home Assistant > Ajustes > Dispositivos y servicios.
2. Anade o reconfigura la integracion ESPHome para `m5-stack-fire.local` o la IP del dispositivo.
3. Cuando Home Assistant pida la encryption key, pega la misma clave.
4. Si había una integración anterior sin cifrado o con otra clave, reconfigurala o eliminala y vuelve a anadirla.

![Captura de la pantalla](images/screenshot.jpeg)

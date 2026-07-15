# ESPHome Windrose para M5Stack Fire

Uso de un M5Stack Fire para presentar los datos de una veleta. 

Diseño basado en la tarjeta de Home Assistant https://github.com/aukedejong/lovelace-windrose-card

## 1. Cambia las entidades

En `m5-stack-fire-windrose-live.yaml`, ajusta estas dos lineas:

```yaml
substitutions:
  wind_direction_entity: sensor.ecowitt_wind_direction
  wind_speed_entity: sensor.ecowitt_wind_speed
```

Usa los `entity_id` reales de Home Assistant.

## 2. Empareja el dispositivo con Home Assistant

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

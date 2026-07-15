# ESPHome Windrose para M5Stack Fire

Este arranque evita el componente externo por ahora. Usa sensores de Home Assistant y dibuja una pantalla nativa en el M5Stack Fire:

- Direccion actual del viento.
- Velocidad actual en nudos.
- Rosa/compas con 16 radios.
- Titulo `Wind Vane` centrado.
- Periodo con hora de primera y ultima muestra almacenada.
- Barra Beaufort vertical a la derecha.
- Flecha roja exterior para la direccion actual.
- Flecha roja sobre la barra Beaufort para la intensidad actual.
- Segmentos historicos coloreados por Beaufort.
- Rosa local de las ultimas 24 horas, con unas 5088 muestras a 17 segundos.
- Actualizacion cada 17 segundos.

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
4. Si habia una integracion anterior sin cifrado o con otra clave, reconfigurala o eliminala y vuelve a anadirla.

## 3. Prueba primero sin `external_components`

Tu YAML actual incluye:

```yaml
external_components:
  - source:
      type: git
      url: https://github.com/francisperea/esphome-windrose
      ref: main
    components: [windrose]
```

Para esta primera version no hace falta. Es mejor quitarlo hasta que la pantalla live compile y funcione.

## 4. Sobre la rosa historica

La tarjeta Lovelace calcula/consulta historicos para periodos como `Today`, `Last 7 days` y `Last 30 days`. Eso no conviene hacerlo dentro del M5Stack Fire.

Para una ventana de 24 horas sigue siendo razonable hacerlo localmente en el M5Stack Fire. Con lecturas cada 17 segundos son unas 5088 muestras.

Segun los logs aportados, la build usaba 53180 bytes de DRAM y quedaban 127556 bytes libres. Esta version guarda 24 horas usando bytes para sector/Beaufort y `uint32_t` para timestamps, alrededor de 30 KB extra.

La version actual usa una ventana circular local:

- Guarda sector de direccion, Beaufort y hora de cada muestra.
- Mantiene hasta 5088 muestras.
- Dibuja 16 radios proporcionales a la frecuencia de cada direccion.
- Apila colores dentro de cada radio segun los Beaufort observados en ese sector.

Si se quiere evolucionar, hay dos opciones:

1. Guardar una ventana circular con direccion y velocidad de las ultimas 5088 lecturas. Es flexible y permite recalcular la rosa.
2. Acumular directamente 16 sectores de direccion y 13 rangos Beaufort. Es mas ligero, pero hay que restar correctamente las muestras que salen de la ventana de 24 horas.

Si algun dia vuelves a querer dia/semana/mes, la arquitectura recomendable seria:

1. Home Assistant, WeeWX, Node-RED o un script calcula los 16 sectores de direccion y los rangos de velocidad.
2. Publica un JSON por MQTT con los porcentajes ya agregados.
3. ESPHome solo consume ese JSON y dibuja la rosa.

Ejemplo de JSON futuro:

```json
{
  "period": "today",
  "direction_deg": 89,
  "speed_knots": 4.28,
  "bins": [0, 1, 4, 8, 12, 9, 3, 2, 0, 0, 0, 1, 5, 10, 7, 2]
}
```

Ese es el punto en el que si tiene sentido crear el componente externo `windrose`.

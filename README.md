# 🌿 Panel de Alertas de Polen en Home Assistant
Autor: Pedro Miralles
Última versión: Julio 2025   
Fuente de datos: [eXPerience83/pollenlevels](https://github.com/eXPerience83/pollenlevels).

## 📊 Introducción

Este panel muestra el nivel de polen para distintas zonas de España  y proporciona recomendaciones por tipo: 🌳 Árboles, 🌱 Gramíneas y 🌾 Maleza. Está basado en sensores en tiempo real del proyecto pollenlevels y diseñado para ser visual, dinámico y funcional con estilo tipo tarjeta meteorológica personalizada.

### 📊 Fuente de datos: 

Los niveles de polen provienen del proyecto libre y actualizado [pollenlevels](https://github.com/eXPerience83/pollenlevels), que ofrece sensores compatibles con Home Assistant para distintas zonas de España. 
Esta integración forma parte del repositorio predeterminado de HACS . No es necesario agregarla manualmente como repositorio personalizado ; simplemente busque "Niveles de polen" directamente en HACS e instálela.

## 🧰 Requisitos

### Para integrarlos:

1. Instala `pollenlevels` como integración personalizada vía HACS.
2. Abre **HACS → Integraciones** en Home Assistant.
3. Haz clic en **Explorar y descargar repositorios** (ícono 🔍).
4. Busca *niveles de polen* o simplemente haz clic en la insignia de arriba para abrirla directamente.
5. Haz clic en **Descargar** y sigue las instrucciones.
6. Reinicia o vuelve a cargar Home Assistant cuando se te solicite.

### ⚙️ Configuración

En `configuration.yaml`, configura tu zona (por ejemplo: *Illes Balears*) e ingresa:

- Clave API de Google  
- Ubicación (completada automáticamente desde la configuración de HA)  
- Intervalo de actualización (horas)  
- Código de idioma (por ejemplo: `en`, `es`, `de`, `fr`, `uk`)  

### 📡 Sensores requeridos

Asegúrate de tener disponibles los siguientes sensores:

- `sensor.arbol`
- `sensor.gramineas`
- `sensor.gramineas_2`

> Todos devuelven valores del 1 al 5, donde 1 es riesgo muy bajo y 5 riesgo muy

## 🧰 Requisitos para el panel visual

Instala los siguientes complementos en **HACS → Frontend**:

- `button-card`: para crear tarjetas dinámicas y estilizadas.
- `card-mod`: para aplicar estilos CSS avanzados.
- `bubble-card`: *opcional*, para separadores decorativos.

💡 **Nota:** Reinicia Home Assistant tras cada instalación para aplicar los cambios.

## 🧪 Crear sensor resumen — nivel máximo

Añade lo siguiente en `configuration.yaml` o `template.yaml`:

```yaml
template:
  - sensor:
      - name: max_polen_nivel
        unique_id: max_polen_nivel
        state: >
          {{ [states('sensor.arbol')|int,
              states('sensor.gramineas')|int,
              states('sensor.gramineas_2')|int] | max }}
```
Esto calcula el valor más alto entre los tres sensores.

## 🔔 Automatización de alerta crítica
Recibirás notificación en el móvil si el riesgo alcanza nivel 4 o 5:

```yaml
alias: Alerta de polen crítico
description: ""
triggers:
  - entity_id:
      - sensor.arbol
      - sensor.gramineas
      - sensor.gramineas_2
    above: 2.8
    trigger: numeric_state
conditions: []
actions:
  - data:
      title: ⚠️ ALERTA DE POLEN ⚠️
      message: >
        Riesgo alto detectado en: {% if states('sensor.arbol') | int > 3 %}🌳
        Árboles{% endif %} {% if states('sensor.gramineas') | int > 3 %}, 🌱
        Gramíneas{% endif %} {% if states('sensor.gramineas_2') | int > 3 %}, 🌾
        Maleza{% endif %}
      data:
        tag: polen_alerta
        channel: polen
        importance: high
        persistent: true
        notification_icon: mdi:biohazard
    action: telegram_bot.send_message
mode: single
```

📱 Reemplaza telegram_bot.send_message por el ID de tu móvil o por tu bot de telegram (ejemplo: mobile_app_pixel_7).

## 🎨 Panel Lovelace — YAML 
Ve a Dashboard → editar vista → YAML y pega:

```yaml
type: vertical-stack
cards:
  - type: markdown
    content: |
      ## 🌸 Alertas de Polen
      Mantente protegido con recomendaciones adaptadas a cada nivel.
    style: |
      ha-card {
        background: linear-gradient(90deg, #E3F2FD, #FCE4EC);
        color: #333;
        border-radius: 12px;
        text-align: center;
        padding: 16px;
        font-size: 18px;
      }

  # Semáforo resumen
  - type: custom:button-card
    entity: sensor.max_polen_nivel
    name: 🚦 Riesgo General de Polen
    show_icon: false
    icon: mdi:traffic-light-outline
    show_state: true
    tap_action: { action: none }
    styles:
      card:
        - border-radius: 12px
        - padding: 6px
        - background: |
            [[[
              let n = states['sensor.max_polen_nivel'].state;
              if (n == '1') return 'linear-gradient(135deg, #A8E05F, #B0E57D)';
              if (n == '2') return 'linear-gradient(135deg, #FDD64B, #FFD97A)';
              if (n == '3') return 'linear-gradient(135deg, #FF9B57, #FFA87D)';
              if (n == '4') return 'linear-gradient(135deg, #FE6A69, #FF8888)';
              return 'linear-gradient(135deg, #A97ABC, #BA8FCD)';
            ]]]
      name: [font-size: 18px, font-weight: bold, color: black]
      state: [font-size: 14px, color: black, white-space: normal, text-align: left]
    state_display: |
      [[[
        let n = states['sensor.max_polen_nivel'].state;
        if (n == '1') return "🟢 Muy Bajo\nAmbiente libre de polen";
        if (n == '2') return "🟢 Bajo\nSin restricciones";
        if (n == '3') return "🟡 Medio\nPrecaución moderada";
        if (n == '4') return "🔴 Alto\nEvitar exposición";
        return "🚨 Muy Alto\nMedidas urgentes recomendadas";
      ]]]

  - type: custom:bubble-card
    card_type: separator

  # 🌳 Árboles
  - type: custom:button-card
    entity: sensor.arbol
    name: 🌳 Árboles
    show_icon: false
    show_state: true
    tap_action: { action: none }
    styles:
      card:
        - border-radius: 12px
        - padding: 6px
        - background: |
            [[[
              let n = states['sensor.arbol'].state;
              if (n == '1') return 'linear-gradient(135deg, #A8E05F, #B0E57D)';
              if (n == '2') return 'linear-gradient(135deg, #FDD64B, #FFD97A)';
              if (n == '3') return 'linear-gradient(135deg, #FF9B57, #FFA87D)';
              if (n == '4') return 'linear-gradient(135deg, #FE6A69, #FF8888)';
              return 'linear-gradient(135deg, #A97ABC, #BA8FCD)';
            ]]]
      name: [font-size: 18px, font-weight: bold, color: black]
      state: [font-size: 14px, color: black, white-space: normal, text-align: left]
    state_display: |
      [[[
        let n = states['sensor.arbol'].state;
        if (n == '1') return "🟢 Muy Bajo\n• Sin riesgo\n• Disfruta al aire libre";
        if (n == '2') return "🟢 Bajo\n• Actividades normales\n• Ventilación habitual";
        if (n == '3') return "🟡 Medio\n• Evita salir temprano y al anochecer\n• Lava cara y manos al volver";
        if (n == '4') return "🔴 Alto\n• Permanece en interiores\n• Usa gafas de sol y mascarilla";
        return "🚨 Muy Alto\n• Cierra ventanas\n• Usa purificador\n• Consulta si hay síntomas fuertes";
      ]]]

  # Repite igual para 🌱 Gramíneas y 🌾 Maleza, cambiando el sensor
```

## 💡 Personalizaciones adicionales

- **Separadores**: Puedes usar `━`, `───`, `⋯` para estilo visual entre secciones.
- **Ocultar si todo está en nivel bajo**: Añade `visibility:` para mostrar solo si el riesgo > `1`.
- **Expansión al hacer clic**: Usa `tap_action` para expandir texto o mostrar más detalles.
- **Sonido de alerta**: Puedes combinarlo con `media_player.play_media` para activar sonidos.

---

## 🪄 Resultado final

Un panel que:

- ✅ Muestra resumen general y desglosa tipos de polen  
- ✅ Cambia de fondo automáticamente por riesgo  
- ✅ Da recomendaciones personalizadas  
- ✅ Te avisa si sube el nivel  
- ✅ Es bonito, claro y elegante

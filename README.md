# Proyecto B.A.C. (Base de Agua y Control) 🌊📡

Sistema IoT de monitoreo hídrico en tiempo real diseñado para la medición, registro y alerta automática de nivel de agua y turbidez mediante arquitectura ESP32 y servicios backend.

---

## 🛠️ Arquitectura del Sistema

### Hardware & Embebidos
- **Microcontrolador:** ESP32 V3.0
- **Sensores:** 
  - Nivel de Agua: Sensor Ultrasónico HC-SR04 (Filtrado por promedio móvil)
  - Calidad de Agua: Sensor de Turbidez Óptica (Conversión ADC 12 bits a NTU)
- **Periféricos & Módulos:**
  - Reloj de Tiempo Real (RTC): DS3231 (I2C)
  - Almacenamiento Local: Módulo microSD (SPI) en formato CSV
  - Visualización Local: Pantalla OLED SSD1306 128x64 (I2C)

### Software & Backend
- **Firmware:** C++ / Arduino Framework (Programación no bloqueante con `millis()`)
- **Backend:** Python con Flask (API REST para recepción de telemetría y disparador de alertas)
- **Base de Datos:** SQLite / PostgreSQL

---

## 🚀 Características Principales

1. **Adquisición sin bloqueo:** Lectura continua de sensores sin interrumpir la transmisión de datos.
2. **Filtrado de Señal:** Algoritmo de supresión de ruido acústico para lecturas estables de distancia.
3. **Resiliencia de Datos:** Almacenamiento local en tarjeta microSD ante eventuales caídas de red.
4. **Sistema de Alertas:** Clasificación en tiempo real de niveles (Bajo, Medio, Alto) y umbrales de turbidez.

---


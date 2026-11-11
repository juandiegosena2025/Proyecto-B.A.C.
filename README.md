<img width="1280" height="960" alt="IMG-20261110-WA0005" src="https://github.com/user-attachments/assets/745da983-ee52-4fa2-a59f-bb9342115015" />
B.A.C. - Base de Agua y Control

Sistema IoT de Monitoreo Hídrico en Tiempo Real
**Validado en campo | SENA CTMAE / Red Tecnoparque Nodo Medellín | Iniciativa MAX CAMPO**

![ESP32](https://img.shields.io/badge/ESP32-C%2B%2B-blue)
![Estado](https://img.shields.io/badge/Estado-Validado_en_Campo-green)
![Versión](https://img.shields.io/badge/Versión-1.0-orange)
![Licencia](https://img.shields.io/badge/Licencia-MIT-lightgrey)

> **El problema:** Las PTAP, alcaldías y juntas de agua no saben en tiempo real si un tanque se está llenando, si se va a desbordar o si el agua está llegando turbia. Hoy tienen que ir a verlo.
>
> **La solución:** Un nodo IoT de bajo costo que mide cada segundo el nivel y la turbidez, lo muestra en pantalla local y lo deja listo para enviarlo a la nube.

---

📸 Prototipo Validado en Campo

![Prototipo B.A.C. en gabinete IP65](docs/foto-prototipo.jpg)
*Figura 1. Prototipo B.A.C. en gabinete IP65 con ESP32, pantalla OLED, sensor ultrasónico HC-SR04 y sensor de turbidez óptico. Validado en Quebrada El Chapolal, Vereda El Brasil, Cisneros, Antioquia.*

---

🎯 ¿Qué hace B.A.C.?

- **Mide nivel de agua** con sensor ultrasónico HC-SR04, con promedio de 5 muestras para eliminar picos por rebote.
- **Mide turbidez** con sensor óptico, conversión ADC 12 bits a voltaje y clasificación cualitativa.
- **Muestra datos en tiempo real** en pantalla OLED SSD1306 128x64 y en Monitor Serial.
- **Opera en campo** en gabinete IP65 con prensaestopas, consumo <120mA a 5V, apto para alimentación solar.
- **Muestreo cada 1 segundo** por `millis()`, sin `delay()` en el loop principal.

🛠️ Hardware - Fase I Implementada

| Componente | Función |
| :--- | :--- |
| **ESP32 V3.0** | Controlador central Dual-Core 240MHz, Wi-Fi / BT |
| **HC-SR04** | Sensor ultrasónico 40kHz para nivel / distancia |
| **Sensor Turbidez Óptico** | Emisor IR + fototransistor, salida analógica |
| **OLED SSD1306** | Pantalla 0.96" 128x64, I2C 0x3C, visualización local |
| **Gabinete IP65 + Prensaestopas** | Protección contra humedad, polvo y salpicaduras |
| **Multímetro UNI-T UT33C+** | Validación de voltajes y continuidad |

> **Fase II - Hoja de Ruta:** RTC DS3231 + MicroSD (log CSV) + Telemetría MQTT / Ubidots / Dashboard con alertas. Arquitectura preparada, no incluida en v1.0.

🔌 Pinout - ESP32

| Pin ESP32 | Conexión |
| :--- | :--- |
| **GPIO 5** | HC-SR04 TRIG (Salida) - *Corregido: antes GPIO 0, pin de BOOT* |
| **GPIO 16** | HC-SR04 ECHO (Entrada) |
| **GPIO 26** | Sensor Turbidez (ADC1_CH1) |
| **GPIO 21** | OLED SDA (I2C) |
| **GPIO 22** | OLED SCL (I2C) |

📐 Lógica de Medición

**Distancia Ultrasónica:**
Distancia (cm) = (Tiempo (us) * 0.0343) / 2

**Nivel y Porcentaje:**
Nivel (cm) = ALTURA_TOTAL (50.0 cm) - Distancia
Porcentaje (%) = (Nivel / ALTURA_TOTAL) * 100

**Turbidez - ADC 12 bits:**
Voltaje (V) = (Lectura_ADC * 3.3V) / 4095

**Clasificación de Nivel (Corregida v1.0):**
- 0 - 25% = BAJA
- 25 - 50% = MEDIA
- 50 - 75% = ALTA
- 75 - 100% = MUY ALTA

**Clasificación de Turbidez:**
- ADC < 1200 (< 0.96V) = MUY TURBIA
- 1200 - 2200 = TURBIA
- 2200 - 3200 = MEDIA
- >= 3200 (>= 2.57V) = CLARA

> **Nota de Calibración:** La escala de turbidez en v1.0 es cualitativa. Para certificación en NTU se requiere curva con patrones Formazina: `NTU = a*V^2 + b*V + c`

📊 Resultados Reales - Prueba en Campo

Prueba realizada en Quebrada El Chapolal, Cisneros. Tanque de referencia 50.0 cm.

| DIST. (cm) | NIVEL (cm) | LLENADO (%) | ESTADO NIVEL | ADC TURBIDEZ | VOLTAJE | ESTADO TURB. |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 12.4 | 37.6 | 75.2% | MUY ALTA | 3410 | 2.747 V | CLARA |
| 12.5 | 37.5 | 75.0% | MUY ALTA | 3398 | 2.737 V | CLARA |
| 28.1 | 21.9 | 43.8% | MEDIA | 1850 | 1.489 V | TURBIA |
| 42.0 | 8.0 | 16.0% | BAJA | 950 | 0.765 V | MUY TURBIA |

**Hallazgos:**
- El promedio de 5 muestras estabilizó la lectura, eliminando picos por rebote en paredes que se observaban con lectura única.
- Consumo del sistema completo en adquisición activa <120mA a 5V, viable para panel solar + batería Li-Ion.
- Gabinete IP65 aisló exitosamente la electrónica de la humedad en cercanías de la quebrada.

💻 Firmware Completo - v1.0 Corregido

```cpp
/*
 * PROYECTO B.A.C. - Base de Agua y Control v1.0
 * Autores: Daniela Herrera Osorio & Juan Diego Sepúlveda Jiménez
 * Entidad: SENA CTMAE / Red Tecnoparque Nodo Medellín - MAX CAMPO
 * Fecha: 28 agosto de 2026
 * Validado en: Quebrada El Chapolal, Vereda El Brasil, Cisneros, Antioquia
 *
 * DESCRIPCIÓN:
 * Monitoreo de nivel y turbidez en tiempo real con ESP32.
 * Muestreo cada 1000ms por millis(). Lectura ultrasónica con
 * promedio de 5 muestras. Bloqueo acotado a 30ms max en pulseIn.
 *
 * FASE I - IMPLEMENTADA: ESP32 + HC-SR04 + Turbidez + OLED + Serial
 * FASE II - HOJA DE RUTA: RTC DS3231 + MicroSD + MQTT / Ubidots
 */

#include <Arduino.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// OLED SSD1306 - 128x64 pixeles, I2C 0x3C
#define ANCHO_PANTALLA 128
#define ALTO_PANTALLA 64
#define DIRECCION_OLED 0x3C

Adafruit_SSD1306 display(ANCHO_PANTALLA, ALTO_PANTALLA, &Wire, -1);
bool oledDisponible = false;

// PINES - CORRECCIÓN: GPIO 0 NO SE USA, es pin de BOOT. Se migra TRIG a GPIO 5.
const int PIN_TRIG = 5;
const int PIN_ECHO = 16;
const float ALTURA_TOTAL_TANQUE = 50.0; // cm - Calibrar en sitio de instalación
const int PIN_TURBIDEZ = 26; // ADC1_CH1
const int ADC_MAX = 4095;
const float VOLTAJE_ADC = 3.3;
unsigned long tiempoAnterior = 0;
const unsigned long TIEMPO_ACTUALIZACION = 1000; // 1000 ms = 1 segundo

struct LecturasSensores {
  float distancia;
  float nivel;
  float porcentaje;
  String estadoNivel;
  int adcTurbidez;
  float voltajeTurbidez;
  String estadoTurbidez;
};

float leerDistanciaRaw() {
  digitalWrite(PIN_TRIG, LOW);
  delayMicroseconds(2);
  digitalWrite(PIN_TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(PIN_TRIG, LOW);
  long duracion = pulseIn(PIN_ECHO, HIGH, 30000); // Bloqueo acotado max 30ms
  if (duracion == 0) return -1.0;
  return (duracion * 0.0343) / 2.0;
}

float leerDistanciaPromediada() {
  float suma = 0;
  int lecturasValidas = 0;
  for (int i = 0; i < 5; i++) {
    float d = leerDistanciaRaw();
    if (d > 0) { suma += d; lecturasValidas++; }
    delayMicroseconds(500);
  }
  if (lecturasValidas == 0) return -1.0;
  return suma / lecturasValidas;
}

float calcularNivel(float distancia) {
  if (distancia < 0) return -1.0;
  float nivel = ALTURA_TOTAL_TANQUE - distancia;
  return constrain(nivel, 0.0, ALTURA_TOTAL_TANQUE);
}

float calcularPorcentaje(float nivel) {
  if (nivel < 0) return -1.0;
  return constrain((nivel / ALTURA_TOTAL_TANQUE) * 100.0, 0.0, 100.0);
}

// CORRECCIÓN: Clasificación coherente de 4 niveles
String clasificarNivel(float porcentaje) {
  if (porcentaje < 0) return "SIN LECTURA";
  if (porcentaje < 25.0) return "BAJA";
  if (porcentaje < 50.0) return "MEDIA";
  if (porcentaje < 75.0) return "ALTA";
  return "MUY ALTA";
}

int leerADCturbidez() { return analogRead(PIN_TURBIDEZ); }
float convertirVoltaje(int adc) { return (adc * VOLTAJE_ADC) / ADC_MAX; }

String clasificarTurbidez(int adc) {
  if (adc < 1200) return "MUY TURBIA";
  else if (adc < 2200) return "TURBIA";
  else if (adc < 3200) return "MEDIA";
  else return "CLARA";
}

void mostrarCabeceraSerial() {
  Serial.println("=====================================================");
  Serial.println("  PROYECTO B.A.C. - ESP32 V1.0 - BASE DE AGUA Y CONTROL");
  Serial.println("  DIST. | NIVEL | LLENADO% | ESTADO || ADC | VOLTAJE | TURB.");
  Serial.println("=====================================================");
}

void actualizarOLED(const LecturasSensores &d) {
  if (!oledDisponible) return;
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 0);
  display.println("PROYECTO B.A.C. ESP32");
  display.drawLine(0, 9, 128, 9, SSD1306_WHITE);
  display.setCursor(0, 13);
  display.print("Llenado: ");
  if (d.porcentaje >= 0) { display.print(d.porcentaje, 1); display.println("%"); }
  else { display.println("N/A"); }
  display.setCursor(0, 24);
  display.print("Nivel  : ");
  display.println(d.estadoNivel);
  display.drawRect(0, 35, 128, 8, SSD1306_WHITE);
  if (d.porcentaje >= 0) {
    int anchoBarra = map((int)d.porcentaje, 0, 100, 0, 124);
    display.fillRect(2, 37, anchoBarra, 4, SSD1306_WHITE);
  }
  display.setCursor(0, 47);
  display.print("Turbidez: ");
  display.println(d.estadoTurbidez);
  display.setCursor(0, 56);
  display.print("ADC: ");
  display.print(d.adcTurbidez);
  display.print(" | ");
  display.print(d.voltajeTurbidez, 2);
  display.println("V");
  display.display();
}

void setup() {
  Serial.begin(115200);
  pinMode(PIN_TRIG, OUTPUT);
  pinMode(PIN_ECHO, INPUT);
  pinMode(PIN_TURBIDEZ, INPUT);
  analogReadResolution(12);
  if (display.begin(SSD1306_SWITCHCAPVCC, DIRECCION_OLED)) {
    oledDisponible = true;
    display.clearDisplay();
    display.setTextColor(SSD1306_WHITE);
    display.setTextSize(1);
    display.setCursor(15, 20);
    display.println("PROYECTO B.A.C.");
    display.setCursor(10, 35);
    display.println("Iniciando sistema...");
    display.display();
    delay(1500); // Solo en arranque, no en loop
  } else {
    Serial.println("Aviso: OLED no encontrada. Operando por Serial.");
  }
  mostrarCabeceraSerial();
}

void loop() {
  unsigned long tiempoActual = millis();
  if (tiempoActual - tiempoAnterior >= TIEMPO_ACTUALIZACION) {
    tiempoAnterior = tiempoActual;
    LecturasSensores datos;
    datos.distancia = leerDistanciaPromediada();
    datos.nivel = calcularNivel(datos.distancia);
    datos.porcentaje = calcularPorcentaje(datos.nivel);
    datos.estadoNivel = clasificarNivel(datos.porcentaje);
    datos.adcTurbidez = leerADCturbidez();
    datos.voltajeTurbidez = convertirVoltaje(datos.adcTurbidez);
    datos.estadoTurbidez = clasificarTurbidez(datos.adcTurbidez);
    Serial.printf(" %8.1f | %9.1f | %9.1f%% | %12s || %12d | %7.3f V | %s\n",
      datos.distancia, datos.nivel, datos.porcentaje,
      datos.estadoNivel.c_str(), datos.adcTurbidez,
      datos.voltajeTurbidez, datos.estadoTurbidez.c_str());
    actualizarOLED(datos);
  }
}
🚀 Instalación y Uso

1. Requisitos
- Arduino IDE 2.x
- Placa ESP32 instalada (Espressif)
- Librerías: `Adafruit_GFX`, `Adafruit_SSD1306`

2. Cargar Firmware
1. Copia el código de arriba en Arduino IDE como `BAC_v1.0.ino`
2. Selecciona placa: ESP32 Dev Module
3. Sube el código

3. Calibración en Sitio
Edita la constante según la altura real de tu tanque:
const float ALTURA_TOTAL_TANQUE = 50.0; // Cambiar por altura real en cm
4. Ver Datos
Abre el Monitor Serial a *115200 baudios*. Verás:
 12.4 | 37.6 | 75.2% | MUY ALTA || 3410 | 2.747 V | CLARA
🗺️ Hoja de Ruta de Escalabilidad
[FASE I - ACTUAL] Prototipo Validado Local
ESP32 + OLED + HC-SR04 + Turbidez
        |
        v
[FASE II] Almacenamiento y Tiempo Real
RTC DS3231 + MicroSD Log CSV
        |
        v
[FASE III] Calibración NTU
Curva polinomial con patrones Formazina
        |
        v
[FASE IV] Telemetría Cloud
MQTT / HTTPS -> Dashboard Ubidots / AWS IoT + Alertas
*Recomendaciones para Fase Industrial:*
- Instalar tubo de aquietamiento (stilling well) para proteger HC-SR04 de turbulencia y espuma.
- Limpieza periódica de lentes del sensor de turbidez para evitar biofouling.
- En zonas sin Wi-Fi: integrar LoRaWAN o GSM (SIM800L / A7670).

👥 Autores y Acompañamiento

*Autores:*
- *Daniela Herrera Osorio* - ADSO Ficha 3002645
- *Juan Diego Sepúlveda Jiménez* - ADSO Ficha 3002645

*Acompañamiento Técnico:*
- Ing. Fabio Bahos - Red Tecnoparque Medellín
- Instructores: Germán Leal, Camilo González

*Entidad:* Servicio Nacional de Aprendizaje (SENA) - CTMAE / Red Tecnoparque Nodo Medellín
*Lugar y Fecha:* Cisneros, Antioquia, Colombia - 28 agosto de 2026

📄 Licencia

Este proyecto es de código abierto bajo Licencia MIT para fines educativos y de prototipado. Para uso comercial / industrial, contactar a los autores.

📞 Contacto para Piloto Empresarial

¿Quiere instalar un nodo piloto de 15 días en su PTAP o tanque?

*Ofrecemos:* Instalación de nodo B.A.C. + registro histórico + dashboard con alertas tempranas por desbordamiento y turbidez alta.

Contactar vía GitHub Issues o LinkedIn de los autores.

---

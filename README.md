```cpp
const int pinEntrada = 25;
const int duracionBitMs = 100;
const int tamanioMensaje = 8;

volatile int flancos = 0;
int mensajeRecibido[tamanioMensaje];

void IRAM_ATTR contarFlancos() {
  flancos++;
}

void setup() {
  Serial.begin(115200);
  pinMode(pinEntrada, INPUT);
  attachInterrupt(digitalPinToInterrupt(pinEntrada), contarFlancos, RISING);

  Serial.println("=== RECEPTOR PWM SINCRONIZADO ===");
}

void loop() {
  Serial.println("Esperando bit de inicio...");

  while (true) {
    noInterrupts();
    flancos = 0;
    interrupts();

    delay(duracionBitMs * 2);

    noInterrupts();
    int flancosDetectados = flancos;
    interrupts();

    if (flancosDetectados > 4000) {
      Serial.println("¡Inicio detectado!");
      break;
    }
  }

  delay(100);

  Serial.print("Mensaje recibido: ");

  for (int i = 0; i < tamanioMensaje; i++) {
    noInterrupts();
    flancos = 0;
    interrupts();

    delay(duracionBitMs);

    noInterrupts();
    int flancosDetectados = flancos;
    interrupts();

    int umbralFlancos = 930;
    mensajeRecibido[i] = (flancosDetectados > umbralFlancos) ? 1 : 0;

    Serial.print(mensajeRecibido[i]);
    Serial.print(" ");
  }

  Serial.println("\n-------------------------------\n");
  delay(3000);
}

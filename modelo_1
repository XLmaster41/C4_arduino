#include <Adafruit_LiquidCrystal.h>
#include <Keypad.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_LEDBackpack.h>

// Definindo as dimensões do teclado
const byte numRows = 4; // 4 linhas
const byte numCols = 4; // 4 colunas

// Mapeamento do teclado
char keys[numRows][numCols] = {
  {'1', '2', '3', 'A'},
  {'4', '5', '6', 'B'},
  {'7', '8', '9', 'C'},
  {'*', '0', '#', 'D'}
};

// Definindo os pinos das linhas e colunas do teclado
byte rowPins[numRows] = {9, 8, 7, 6}; // Conecte as linhas aos pinos do Arduino
byte colPins[numCols] = {5, 4, 3, 2}; // Conecte as colunas aos pinos do Arduino

// Pinos para o botão e buzzer
int buttonPin = 12; // Botão de iniciar
int switchPin = 10; // Interruptor deslizante para cancelar
int buzzerPin = 11; // Buzzer para alarme
int ledGreenPin = A0; // LED verde para sucesso
int ledRedPin = A1; // LED vermelho para falha

int buttonState = 0;
bool startInput = false; // Flag para iniciar a captura das teclas
bool countingDown = false; // Flag para saber se a contagem regressiva está ativa
bool countdownInterrupted = false; // Flag para saber se a contagem foi interrompida

// Variável para armazenar o número digitado
String inputNumber = "";
int countdownTime = 0; // Tempo de contagem regressiva
unsigned long previousMillis = 0; // Para o controle de tempo
const long interval = 1000; // Intervalo de 1 segundo

// Criação do objeto Keypad
Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, numRows, numCols);

// Configuração do LCD
Adafruit_LiquidCrystal lcd_1(0); // Endereço do LCD, ajuste se necessário

// Configuração do Display de 7 Segmentos
Adafruit_7segment display = Adafruit_7segment();

void setup() {
  pinMode(buttonPin, INPUT_PULLUP); // Configura o botão como entrada com PULLUP
  pinMode(switchPin, INPUT_PULLUP); // Configura o interruptor como entrada
  pinMode(buzzerPin, OUTPUT); // Configura o buzzer como saída
  pinMode(ledGreenPin, OUTPUT); // Configura o LED verde como saída
  pinMode(ledRedPin, OUTPUT); // Configura o LED vermelho como saída

  lcd_1.begin(16, 2); // Inicia o LCD 16x2
  lcd_1.print("Press Btn to start");

  display.begin(0x70); // Endereço I2C do HT16K33 (0x70 é o padrão)
  display.setBrightness(15); // Ajusta o brilho (0-15)
}

void loop() {
  // Lê o estado do botão de início (LOW significa pressionado)
  buttonState = digitalRead(buttonPin);

  // Se o botão for pressionado e ainda não iniciou a entrada
  if (buttonState == LOW && !startInput && !countingDown) {
    startInput = true; // Habilita a captura do teclado
    lcd_1.clear();
    lcd_1.print("Digite o tempo:");
    lcd_1.setCursor(0, 1); // Mover cursor para segunda linha
    delay(300); // Debounce simples
  }

  if (startInput && !countingDown) { // Se o processo foi iniciado e não está contando
    char key = keypad.getKey(); // Lê o valor do teclado

    if (key) { // Se uma tecla foi pressionada
      if (key == 'A' && inputNumber.length() > 0) {
        // Trava o número atual e inicia a contagem regressiva
        countdownTime = inputNumber.toInt(); // Converte o número para int
        countingDown = true;
        lcd_1.clear();
        lcd_1.print("Contagem: ");
        lcd_1.setCursor(0, 1); // Move cursor para a linha da contagem
        previousMillis = millis(); // Inicia a contagem de tempo
      } else if (key == 'B') {
        // Remove o último caractere se a tecla 'B' for pressionada
        if (inputNumber.length() > 0) {
          inputNumber.remove(inputNumber.length() - 1); // Remove o último caractere
          lcd_1.clear();
          lcd_1.print("Digite o tempo:");
          lcd_1.setCursor(0, 1);
          lcd_1.print(inputNumber); // Atualiza o display com o número atual
        }
      } else {
        // Adiciona o número à string de entrada
        if (key >= '0' && key <= '9' && inputNumber.length() < 3) { // Limita a 3 dígitos
          inputNumber += key;
          lcd_1.setCursor(0, 1); // Define o cursor na posição atual
          lcd_1.print(inputNumber); // Exibe o número no LCD
        }
      }
    }
  }

  if (countingDown) { // Se está contando
    unsigned long currentMillis = millis(); // Obtém o tempo atual

    // Verifica se já passou 1 segundo
    if (currentMillis - previousMillis >= interval) {
      previousMillis = currentMillis; // Atualiza o último tempo

      display.print(countdownTime); // Exibe a contagem regressiva no display de 7 segmentos
      display.writeDisplay(); // Atualiza o display

      countdownTime--; // Decrementa o tempo

      // Verifica se o interruptor foi acionado
      if (digitalRead(switchPin) == LOW) {
        countdownInterrupted = true;
        countingDown = false;
        breakCountdown("Interrompido");
      }

      // Quando o tempo chegar a zero
      if (countdownTime < 0 && !countdownInterrupted) {
        endCountdown();
      }
    }
  }
}

// Função para encerrar a contagem com sucesso
void endCountdown() {
  countingDown = false;
  digitalWrite(ledRedPin, HIGH); // Acende o LED vermelho
  digitalWrite(buzzerPin, HIGH); // Liga o buzzer
  lcd_1.clear();
  lcd_1.print("Tempo esgotado!");
  delay(30000); // Mantém o alarme por 3 segundos
  digitalWrite(buzzerPin, LOW);
  digitalWrite(ledRedPin, LOW);

  resetSystem(); // Reseta o sistema
}

// Função para interromper a contagem
void breakCountdown(String message) {
  countingDown = false;
  digitalWrite(ledGreenPin, HIGH); // Acende o LED verde
  lcd_1.clear();
  lcd_1.print(message);
  delay(3000); // Mantém o LED verde aceso por 3 segundos
  digitalWrite(ledGreenPin, LOW);

  resetSystem(); // Reseta o sistema
}

// Função para resetar o sistema
void resetSystem() {
  startInput = false;
  countdownInterrupted = false;
  inputNumber = ""; // Limpa o número de entrada
  lcd_1.clear();
  lcd_1.print("Press Btn to start");
}

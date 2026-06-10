# -.-
Робот для перемещения паллет на складе. Команда "Чайники"
Робот для перемещения паллет на складе

Цель проекта — создать автономного робота, предназначенного для автоматизации перемещения тяжёлых паллет на складах, в супермаркетах и производственных цехах. Робот следует по напольной разметке, самостоятельно поднимает паллету и доставляет её в заданную зону.

Ожидаемый продуктовый результат — функционирующий прототип робота, способный:

- двигаться по линии с помощью двух датчиков;
- поднимать паллету вертикальным механизмом без гидравлики;
- выполнять цикл: подъезд → подъём → транспортировка → разгрузка.

Ключевая польза:

- снижение физической нагрузки на сотрудников;
- уменьшение простоев и времени перемещений;
- повышение предсказуемости внутренней логистики.

Стек технологий:

Категория; Технологии; Инструменты
Языки программирования: C++ (Arduino) 
CAD-системы: Компас 3D
Среда разработки: Arduino IDE
Управление версиями: GitHub 

Аппаратная часть (комплектующие):

- Микроконтроллер: Arduino Nano
- Датчики линии: 2 шт., TCRT5000
- Двигатели: 2 мотора-редуктора
- Механизм подъёма: сервопривод + рейка или мотор-редуктор с винтовым приводом
- Питание: батарейный отсек (аккумуляторы)
- Корпус и детали: 3D-печать

Дополнительно: мотор-драйверы, провода, крепёж, защитный корпус.

Программная логика:

- ПИД-регулятор — точное слежение за линией.

- Конечный автомат (FSM):

- поиск линии;

- движение по линии;

- останов перед паллетой;

- подъём паллеты;

- продолжение движения до зоны выгрузки.

- Аварийная остановка и контроль заряда батареи.

Уникальность решения:

- Компактный вертикальный подъём паллеты без гидравлики — дешевле и проще в обслуживании.

- Работа всего с двумя датчиками линии — минимизация затрат при сохранении точности.

- Отказ от дорогой навигации (SLAM, камеры) — надёжность в средах с предсказуемой разметкой.

План внедрения:

- Доработка прототипа и лабораторные испытания.

- Пилот — тест на складе на 1–2 участках (3 месяца).

- Оптимизация — сбор метрик (время цикла, отказоустойчивость).

- Масштабирование — интеграция нескольких роботов с централизованным управлением зарядом и маршрутами.

Последующие действия:
- Доработать прототип.

- Провести пилот на одном складе.

Код Arduino

#include <Servo.h>

// === Пины моторов (только направление) ===
int in1 = 8;
int in2 = 7;
int in3 = 5;
int in4 = 4;

// === Пины датчиков линии ===
#define LINE_LEFT   2
#define LINE_RIGHT  3

// === Серво на D9 ===
Servo servo;
int servoPin = 9;
int servoUp = 95;
int servoDown = 0;

// === Переменные для перекрёстка ===
unsigned long crossTime = 0;
bool atCrossroad = false;
int crossroadStage = 0;

void setup() {
  // Моторы
  pinMode(in1, OUTPUT);
  pinMode(in2, OUTPUT);
  pinMode(in3, OUTPUT);
  pinMode(in4, OUTPUT);
  stopMotors();

  // Датчики линии
  pinMode(LINE_LEFT, INPUT);
  pinMode(LINE_RIGHT, INPUT);

  // Серво
  servo.attach(servoPin);
  servo.write(servoDown);

  delay(3000); // Пауза перед стартом
}

void loop() {
  bool left = digitalRead(LINE_LEFT);    // 0 = чёрный, 1 = белый
  bool right = digitalRead(LINE_RIGHT);

  // Проверка на перекрёсток
  if (left == 0 && right == 0 && !atCrossroad) {
    atCrossroad = true;
    crossroadStage = 0;
    crossTime = millis();
    stopMotors();
  }

  // Обработка перекрёстка или движение
  if (atCrossroad) {
    handleCrossroad();
  } else {
    followLine(left, right);
  }
}

// === Движение по линии (без ШИМ) ===
void followLine(bool left, bool right) {
  if (left == 1 && right == 1) {
    // Оба на белом — едем прямо
    forward();
  }
  else if (left == 0 && right == 1) {
    // Левый на чёрном — поворачиваем вправо
    turnRight();
  }
  else if (left == 1 && right == 0) {
    // Правый на чёрном — поворачиваем влево
    turnLeft();
  }
  else if (left == 0 && right == 0) {
    // Оба на чёрном (проезжаем)
    forward();
  }
}

// === Поворот вправо: левый мотор вперёд, правый стоп ===
void turnRight() {
  digitalWrite(in1, HIGH);
  digitalWrite(in2, LOW);   // Левый вперёд
  digitalWrite(in3, LOW);
  digitalWrite(in4, LOW);   // Правый стоп
}

// === Поворот влево: правый мотор вперёд, левый стоп ===
void turnLeft() {
  digitalWrite(in1, LOW);
  digitalWrite(in2, LOW);   // Левый стоп
  digitalWrite(in3, HIGH);
  digitalWrite(in4, LOW);   // Правый вперёд
}

// === Вперёд ===
void forward() {
  digitalWrite(in1, HIGH);
  digitalWrite(in2, LOW);   // Левый вперёд
  digitalWrite(in3, HIGH);
  digitalWrite(in4, LOW);   // Правый вперёд
}

// === Назад ===
void backward() {
  digitalWrite(in1, LOW);
  digitalWrite(in2, HIGH);  // Левый назад
  digitalWrite(in3, LOW);
  digitalWrite(in4, HIGH);  // Правый назад
}

// === Стоп ===
void stopMotors() {
  digitalWrite(in1, LOW);
  digitalWrite(in2, LOW);
  digitalWrite(in3, LOW);
  digitalWrite(in4, LOW);
}

// === Обработка перекрёстка ===
void handleCrossroad() {
  switch (crossroadStage) {
    case 0:
      // Поднять серво
      servo.write(servoUp);
      crossTime = millis();
      crossroadStage = 1;
      break;

    case 1:
      // Ждать 0.5 сек (чтобы серво успела подняться)
      if (millis() - crossTime > 500) {
        crossTime = millis();
        crossroadStage = 2;
      }
      break;

    case 2:
      // Ждать 5 секунд
      if (millis() - crossTime > 5000) {
        servo.write(servoDown);
        crossTime = millis();
        crossroadStage = 3;
      }
      break;

    case 3:
      // Ждать 0.5 сек (чтобы серво опустилась)
      if (millis() - crossTime > 500) {
        atCrossroad = false;
        crossroadStage = 0;
      }
      break;
  }
}
- Собрать KPI.

- Подготовить коммерческое предложение.

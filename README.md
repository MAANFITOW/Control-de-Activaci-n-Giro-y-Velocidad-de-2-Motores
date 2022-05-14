# Control-de-Activacion-Giro-y-Velocidad-de-2-Motores
Este es el código empleado para la resolución el examen parcial de la materia Sistemas Embebidos 2.

![](https://i.ibb.co/MNQJdRk/Examen.jpg)

````cpp
#include <Arduino.h>
// NOTA: Todos los #define referencian en numero al pin utilizado
// PLACA: ESP32 Dev Kit V1
// M1: Motor Izquierdo
// M2: Motor Derecho

// ENTRADAS
#define ps 13
#define p1 12
#define p2 33
#define p3 32
#define pot 26

// SALIDAS
#define M11 5
#define M12 18
#define M21 19
#define M22 21
#define PWM 23

// VARIABLES GLOBALES
// Determinaran la activacion, sentido y velocidad de los motores
float vel = 0;
bool m1 = true;
bool m2 = true;
bool sentido;

// Parametros para el pin analogico que determina la velocidad
const int frec = 1000;
const int canal = 0;
const int res = 8;

// FUNCIONES DE CADA BOTON
void metodo_ps() {
	sentido = !sentido;
	Serial.println("El sentido fue cambiado");
}
void metodo_p1() {
	m1 = true;
	m2 = false;
	Serial.println("p1");
}
void metodo_p2() {
	m1 = 1;
	m2 = 1;
	Serial.println("p2");
}
void metodo_p3() {
	m1 = false;
	m2 = true;
	Serial.println("p3");
}

// FUNCION DE CONTROL DE MOTORES
void motor(float vel, bool m1, bool m2, bool sentido) {

	// Definicion del Sentido
	int valor1 = 0;
	int valor2 = 1;

	if (sentido) {
		valor1 = 1;
		valor2 = 0;
	} else {
		valor1 = 0;
		valor2 = 1;
	}

	// Motor 1
	if (m1) {
		digitalWrite(M11, valor1);
		digitalWrite(M12, valor2);
		ledcWrite(canal, vel);
	} else {
		digitalWrite(M11, 0);
		digitalWrite(M12, 0);
	}

	// Motor 2
	if (m2) {
		digitalWrite(M21, valor1);
		digitalWrite(M22, valor2);
		ledcWrite(canal, vel);
	} else {
		digitalWrite(M21, 0);
		digitalWrite(M22, 0);
	}
}


void setup() {

	// DECLARACION DE ENTRADAS
	pinMode(p1, INPUT);
	attachInterrupt(digitalPinToInterrupt(p1),metodo_p1, RISING);
	pinMode(p2, INPUT);
	pinMode(p3, INPUT);
	attachInterrupt(digitalPinToInterrupt(p3),metodo_p3, RISING);
	pinMode(pot, INPUT);
	//DECLARACION DE SALIDAS
	pinMode(M11, OUTPUT);
	pinMode(M12, OUTPUT);
	pinMode(M21, OUTPUT);
	pinMode(M22, OUTPUT);

	pinMode(PWM, OUTPUT);
	ledcSetup(canal, frec, res);
	ledcAttachPin(PWM, canal);
	analogSetAttenuation(ADC_11db);

	Serial.begin(115200);

}

void loop() {

	// RAZON DE LA VELOCIDAD 255 / 4095 = 0.06227
	vel = analogRead(pot) * 0.06227;

	// Se lee el estado del pulsador PS
	if (digitalRead(ps) == 1) {
		delay(100);
		if (digitalRead(ps) == 1) {
			metodo_ps();
		}
	}

	// Se lee el estado del pulsador P2
	if (digitalRead(p2) == 1) {
		delay(100);
		if (digitalRead(p2) == 1) {
			metodo_p2();
		}
	}

	// Se hace la magia del motor
	motor(vel, m1, m2, sentido);

}

````

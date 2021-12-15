# BOT-EXPLORER-V1

Títol i Autor 

/* Project: Programa Final */

/* Author: Estela Salas */
Llibreries 
#include <math.h>
#include "IRremote.h"
#include "ABLocks_TimerFreeTone.h"
Variables 
double Temperature;
double LlumLDR;
double Distance;
double RemoteControl;
double Frequency;
Receptor del comandament a distància
IRrecv ir_rx(11);
decode_results ir_rx_results;
unsigned long task_time_ms=0;
Variable de la temperatura unida al sensor NTC de captació de la temperatura
double fnc_3dbot_ntc(int _rawval){
	long Resistance;
	double Temp;
	Resistance=6000.0*((1024.0/_rawval) - 1);
	Temp = log(Resistance);
	Temp = 1 / (0.001129148 + (0.000234125 * Temp) + (0.0000000876741 * Temp * Temp * Temp));
	Temp = Temp - 273.15;
	return Temp;
}
Instruccions per al sensor NTC i conseqüències segons la temperatura que en capta, representades per l’activació dels LEDs vermell o verd.
void NTC() {
	Temperature = fnc_3dbot_ntc(analogRead(A3));
	if ((Temperature >= 25)) {
		analogWrite(6,255);
	}
	else {
		analogWrite(6,0);
	}
	if ((Temperature < 25)) {
		analogWrite(3,255);
	}
	else {
		analogWrite(3,0);
	}
}
Instruccions per al sensor de llum LDR i conseqüències segons la llum que en capta, representades per l’activació del LED groc.
void light() {
	LlumLDR = analogRead(A2);
	if ((LlumLDR < 100)) {
		analogWrite(5,255);
	}
	else {
		analogWrite(5,0);
	}
}
Relació del moviment dels motor segons la tecla del comandament a distància, per a cada una d’aquestes en especific. 
unsigned long fnc_3dbot_irrx()
{
	bool decoded=false;
	if( ir_rx.decode(&ir_rx_results))
	{
		decoded=true;
		ir_rx.resume();
	}
	 if(decoded) return ir_rx_results.value; else return 0;
 }

void fnc_3dbot_motor(int _m, int _dir, int _speed){
	int pin1=7;
	int pin2=8;
	int pin3=9;
	switch(_m){
		case 0: //left
			pin1=7;
			pin2=8;
			pin3=9;
			break;
		case 1: //right
			pin1=12;
			pin2=13;
			pin3=10;
			break;
	}
	switch(_dir){
		case 0: //stop
			analogWrite(pin3,0);
			break;
		case 1: //forward
			digitalWrite(pin1, LOW);
			digitalWrite(pin2, HIGH);
			analogWrite(pin3,_speed);
			break;
		case 2: //backward
			digitalWrite(pin1, HIGH);
			digitalWrite(pin2, LOW);
			analogWrite(pin3,_speed);
			break;
	}
}

void fnc_3dbot_move(int _t, int _speed){
	switch(_t){
		case 0: //stop
			digitalWrite(9, LOW);
			digitalWrite(10, LOW);
			break;
		case 1: //fw
			digitalWrite(7, LOW);
			digitalWrite(8, HIGH);
			digitalWrite(12, LOW);
			digitalWrite(13, HIGH);
			analogWrite(9,_speed);
			analogWrite(10,_speed);
			break;
		case 2: //bw
			digitalWrite(7, HIGH);
			digitalWrite(8, LOW);
			digitalWrite(12, HIGH);
			digitalWrite(13, LOW);
			analogWrite(9,_speed);
			analogWrite(10,_speed);
			break;
		case 3: //left
			digitalWrite(7, LOW);
			digitalWrite(8, HIGH);
			digitalWrite(12, LOW);
			digitalWrite(13, HIGH);
			analogWrite(9,0);
			analogWrite(10,_speed);
			break;
		case 4: //right
			digitalWrite(7, LOW);
			digitalWrite(8, HIGH);
			digitalWrite(12, LOW);
			digitalWrite(13, HIGH);
			analogWrite(9,_speed);
			analogWrite(10,0);
			break;
		case 5: //rotate left
			digitalWrite(7, HIGH);
			digitalWrite(8, LOW);
			digitalWrite(12, LOW);
			digitalWrite(13, HIGH);
			analogWrite(9,_speed);
			analogWrite(10,_speed);
			break;
		case 6: //rotate right
			digitalWrite(7, LOW);
			digitalWrite(8, HIGH);
			digitalWrite(12, HIGH);
			digitalWrite(13, LOW);
			analogWrite(9,_speed);
			analogWrite(10,_speed);
			break;
		case 7: //left bw
			digitalWrite(7, HIGH);
			digitalWrite(8, LOW);
			digitalWrite(12, HIGH);
			digitalWrite(13, LOW);
			analogWrite(9,0);
			analogWrite(10,_speed);
			break;
		case 8: //right bw
			digitalWrite(7, HIGH);
			digitalWrite(8, LOW);
			digitalWrite(12, HIGH);
			digitalWrite(13, LOW);
			analogWrite(9,_speed);
			analogWrite(10,0);
			break;
	}
}
Instruccions per als motors per a la direcció i la velocitat d’aquests segon la tecla del comandament a distància.
void Movement() {
	if((millis()-task_time_ms)>=1000){
		task_time_ms=millis();
		Serial.println((unsigned long)fnc_3dbot_irrx());
	}
	RemoteControl = (unsigned long)fnc_3dbot_irrx();
	if ((RemoteControl == 16736925)) {
		fnc_3dbot_motor(1,1,200);
		fnc_3dbot_motor(0,1,200);
	}
	if ((RemoteControl == 16720605)) {
		fnc_3dbot_move(3,220);
	}
	if ((RemoteControl == 16754775)) {
		fnc_3dbot_motor(1,2,200);
		fnc_3dbot_motor(0,2,200);
	}
	if ((RemoteControl == 16761405)) {
		fnc_3dbot_move(4,220);
	}
	if ((RemoteControl == 16712445)) {
		fnc_3dbot_move(0,220);
	}
}
double fnc_3dbot_distance(int _t, int _e){
	unsigned long dur=0;
	digitalWrite(_t, LOW);
	delayMicroseconds(5);
	digitalWrite(_t, HIGH);
	delayMicroseconds(10);
	digitalWrite(_t, LOW);
	dur = pulseIn(_e, HIGH, 18000);
	if(dur==0)return 999.0;
	return (dur/57);
}
Instruccions per a l’activació del brunzidor i la freqüència que emet segons la distància indicada respecte als obstacles.
void Buzzer() {
	Distance = fnc_3dbot_distance(4,2);
	if ((Distance == 15)) {
		fnc_3dbot_move(0,220);
	}
	Frequency = 0;
	if ((Distance == 50)) {
		TimerFreeTone(A0,261.63,1000);
	}
	if ((Distance == 35)) {
		TimerFreeTone(A0,277.18,1000);
	}
	if ((Distance < 20)) {
		TimerFreeTone(A0,0,100);
	}
	if ((Distance == 20)) {
		TimerFreeTone(A0,293.66,1000);
	}
}
Els  bauds, la velocitat de transmissió de les senyals expressats en símbols per segons, amb què s’inicia el programa. 

Els pins d’entrada dels sensors, del motor o del receptor de infraroigs.
void setup()
{
  	Serial.begin(9600);
	Serial.flush();
	while(Serial.available()>0)Serial.read();

pinMode(A3, INPUT);
pinMode(6, OUTPUT);
pinMode(3, OUTPUT);
pinMode(A2, INPUT);
pinMode(5, OUTPUT);
ir_rx.enableIRIn();
pinMode(7, OUTPUT);
pinMode(8, OUTPUT);
pinMode(9, OUTPUT);
pinMode(10, OUTPUT);
pinMode(12, OUTPUT);
pinMode(13, OUTPUT);
pinMode(4, OUTPUT);
pinMode(2, INPUT);
pinMode(A0, OUTPUT);
}
Bucle al qual es troba sotmès el programa.
void loop()
{
  	NTC();
  	Movement();
  	Buzzer();
  	light();
}




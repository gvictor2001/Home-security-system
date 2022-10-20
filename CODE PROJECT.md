// Home-security-system
const unsigned int TRIG_PIN=13;
const unsigned int ECHO_PIN=12;
const unsigned int BAUD_RATE=9600;

#include <Wire.h> 
#include <Keypad.h>
#include <Servo.h>
Servo myservo;

#define lungime_parola 5 

//int signalPin = 12;
int arm=A0,deza=A1;
int buzzerA=A2;
int buzz=8;
int dezarmat=1;
int buttonChange=10;
int change=0;
int pos = 0;
int cancelbutton=9;
int gresit=0;
int servom=11;


char Data[lungime_parola]; 
char Parola[lungime_parola] = "1234"; 
char newpass[lungime_parola];
char newpassverif[lungime_parola];
char verifpass[lungime_parola];
byte data_count = 0, master_count = 0,newcount=0,newcountverif=0,verifcount=0;
bool Pass_is_good;
char customKey;
int buttonState=0,cancelbuttonState=0;

const byte ROWS = 3;
const byte COLS = 3;

char hexaKeys[ROWS][COLS] = {
  {'1', '2', '3'},
  {'4', '5', '6'},
  {'7', '8', '9'}
};

byte rowPins[ROWS] = {5, 6, 7};
byte colPins[COLS] = {2, 3, 4};

Keypad customKeypad = Keypad(makeKeymap(hexaKeys), rowPins, colPins, ROWS, COLS);  //declarare tastatura


void setup(){
  Serial.begin(9600);
  pinMode(arm,OUTPUT);
  pinMode(deza,OUTPUT);
  digitalWrite(deza,HIGH);
  pinMode(buzzerA, OUTPUT); //buzzer
  //Serial.println("Enter Password:");
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(buttonChange, INPUT);
  myservo.attach(servom);
  opendoor();
}

void loop(){

  buttonState = digitalRead(buttonChange);
  cancelbuttonState=digitalRead(cancelbutton);
  if(buttonState == HIGH && dezarmat==1)
  {
    change=1;
    digitalWrite(deza,HIGH);
    digitalWrite(arm,HIGH);
    Serial.print("Change");
    digitalWrite(buzzerA,HIGH);
    delay(100);
    digitalWrite(buzzerA,LOW);
    
  }
  if(change==1 && dezarmat==1)
  {
    //changepass();
    passinit();
  }
  else if(change==2)
  {
    //passverif();
    changepass();
  }
  else if(change==3)
  {
    passverif();
  }
  else
  {
    parola();
  }
  if(cancelbuttonState == HIGH)
  {
    
    //Serial.print("Cancel");
    change=0;
    clearData();
    clearData1();
    clearData2();
    clearintverif();
    digitalWrite(buzzerA,HIGH);
    delay(100);
    digitalWrite(buzzerA,LOW);
    digitalWrite(deza,LOW);
    digitalWrite(arm,LOW);
    if(dezarmat==1)
      digitalWrite(deza,HIGH);
    else
      digitalWrite(arm,HIGH);
  }
  
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);
  

 const unsigned long duration= pulseIn(ECHO_PIN, HIGH);
 int distance= duration/29/2;
 if(duration==0){
   Serial.println("Warning: no pulse from sensor");
   } 
  else{
      //Serial.print("distance to nearest object:");
      Serial.println(distance);
      //Serial.println(" cm");
      if(distance>3 && distance<14 && dezarmat==0){
          //digitalWrite(buzzerA,HIGH);
          siren();
      }
      else{
        //digitalWrite(buzzerA,LOW);
        }
  }
}


void clearData1(){
  while(newcount !=0){
    newpass[newcount--] = 0; 
  }
  return;
}
void clearData(){
  while(data_count !=0){
    Data[data_count--] = 0; 
  }
  return;
}
void clearintverif(){
  while(verifcount !=0){
    verifpass[verifcount--] = 0; 
  }
  return;
}
void opendoor()
{
  
  for (pos = 80; pos >= 0; pos -= 1) { 
    myservo.write(pos);             
    delay(15); 
  }
}
void closedoor()
{
  for (pos = 0; pos <= 80; pos += 1) { 
    myservo.write(pos);            
    delay(15);                   
  }
}
void siren()
{
  digitalWrite(buzzerA,HIGH);
  for(int i=0;i<5;i++){
  delay(300);
noTone(buzz);
tone(buzz,494,500);
delay(300);
noTone(buzz);
tone(buzz,523,300);
delay(200);
noTone(buzz);
    digitalWrite(buzzerA,LOW);
  }
}

void parola()
{
  customKey = customKeypad.getKey();
  if (customKey){
    Data[data_count] = customKey;  
    Serial.print(Data[data_count]); 
    data_count++; 
    digitalWrite(buzzerA,HIGH);
      delay(100);
      digitalWrite(buzzerA,LOW);
    }

  if(data_count == lungime_parola-1){

    if(!strcmp(Data, Parola)){
      if(dezarmat==1)
      {
        digitalWrite(arm,HIGH);
        digitalWrite(deza,LOW);
        digitalWrite(buzzerA,HIGH);
        delay(50);
        digitalWrite(buzzerA,LOW);
        dezarmat=0;
        closedoor();
      }
      else
      {
        digitalWrite(arm,LOW);
        digitalWrite(deza,HIGH);
        digitalWrite(buzzerA,HIGH);
        delay(50);
        digitalWrite(buzzerA,LOW);
        dezarmat=1;
        opendoor();
      }
      Serial.print("Correct");
      }
    else
    {
      Serial.print("Incorrect");
      digitalWrite(arm,HIGH);
      digitalWrite(deza,HIGH);
      digitalWrite(buzzerA,HIGH);
      delay(500);
      digitalWrite(arm,LOW);
      digitalWrite(deza,LOW);
      digitalWrite(buzzerA,LOW);
      delay(500);
      digitalWrite(arm,HIGH);
      digitalWrite(deza,HIGH);
      digitalWrite(buzzerA,HIGH);
      delay(500);
      gresit++;
      if(gresit==3)
      {
        gresit=0;
        siren();
      }
      digitalWrite(arm,LOW);
      digitalWrite(deza,LOW);
      digitalWrite(buzzerA,LOW);
      delay(500);
      
      if(dezarmat==1)
      digitalWrite(deza,HIGH);
      else
      digitalWrite(arm,HIGH);
    }
    clearData();  
  }
}


void passinit()
{
  customKey = customKeypad.getKey();
  if (customKey){
    verifpass[verifcount] = customKey;  
    verifcount++; 
    digitalWrite(buzzerA,HIGH);
    delay(100);
    digitalWrite(buzzerA,LOW);
    }
    if(verifcount == lungime_parola-1){
      //change=2;
      
      if(!strcmp(Parola, verifpass))
      {
        delay(500);
        digitalWrite(deza,LOW);
      digitalWrite(arm,LOW);
      delay(500);
      digitalWrite(deza,HIGH);
      digitalWrite(buzzerA,HIGH);
      delay(500);
      digitalWrite(deza,LOW);
      digitalWrite(buzzerA,LOW);
      delay(500);
      digitalWrite(deza,HIGH);
      delay(500);
      digitalWrite(deza,LOW);
       digitalWrite(deza,HIGH);
      digitalWrite(arm,HIGH);
      changepass();
      change=2;
      
        }
        else
        {
          change=0;
          clearintverif();
          delay(500);
      digitalWrite(arm,HIGH);
      digitalWrite(buzzerA,HIGH);
      delay(500);
      digitalWrite(arm,LOW);
      digitalWrite(buzzerA,LOW);
      delay(500);
      digitalWrite(arm,HIGH);
      digitalWrite(buzzerA,HIGH);
      delay(500);
      digitalWrite(arm,LOW);
      digitalWrite(buzzerA,LOW);
        }
    }
}

void passverif()
{
  customKey = customKeypad.getKey();
  if (customKey){
    newpassverif[newcountverif] = customKey;  
    Serial.print(newpassverif[newcountverif]); 
    newcountverif++; 
    digitalWrite(buzzerA,HIGH);
    delay(100);
    digitalWrite(buzzerA,LOW);
    }
    if(newcountverif == lungime_parola-1){
      change=0;
      digitalWrite(deza,LOW);
      digitalWrite(arm,LOW);
      if(!strcmp(newpass, newpassverif))
      {
      delay(500);
      digitalWrite(deza,HIGH);
      digitalWrite(buzzerA,HIGH);
      delay(500);
      digitalWrite(deza,LOW);
      digitalWrite(buzzerA,LOW);
      delay(500);
      digitalWrite(deza,HIGH);
      delay(500);
      digitalWrite(deza,LOW);
      /*digitalWrite(buzzerA,HIGH);
      delay(100);
      digitalWrite(buzzerA,LOW);*/
      for(int i=0;i<lungime_parola;i++)
        Parola[i]=newpass[i];
      }
      else
      {
      delay(500);
      digitalWrite(arm,HIGH);digitalWrite(buzzerA,HIGH);
      delay(500);
      digitalWrite(arm,LOW);
      digitalWrite(buzzerA,LOW);
      delay(500);
      digitalWrite(arm,HIGH);digitalWrite(buzzerA,HIGH);
      delay(500);
      digitalWrite(arm,LOW);
      digitalWrite(buzzerA,LOW);
      }
      
     if(dezarmat==1){
        digitalWrite(deza,HIGH);
     }
     else
     {
      digitalWrite(arm,HIGH);
     }
     clearData1();
     clearData2();
     clearintverif();
     
    }
}
void clearData2(){
  while(newcountverif !=0){
    newpassverif[newcountverif--] = 0; 
  }
  return;
}
void changepass()
{
  customKey = customKeypad.getKey();
  if (customKey){
    newpass[newcount] = customKey;  
    Serial.print(newpass[data_count]); 
    newcount++; 
    digitalWrite(buzzerA,HIGH);
    delay(100);
    digitalWrite(buzzerA,LOW);
    }
    if(newcount == lungime_parola-1){
      change=3;
      digitalWrite(deza,LOW);
      digitalWrite(arm,LOW);
      delay(500);
      digitalWrite(deza,HIGH);
      digitalWrite(buzzerA,HIGH);
      delay(500);
      digitalWrite(deza,LOW);
      digitalWrite(buzzerA,LOW);
      delay(500);
      digitalWrite(deza,HIGH);
      delay(500);
      digitalWrite(deza,LOW);
       digitalWrite(deza,HIGH);
      digitalWrite(arm,HIGH);
      //for(int i=0;i<lungime_parola;i++)
       // Parola[i]=newpass[i];
     passverif();
     //clearData1();
    }
   
}

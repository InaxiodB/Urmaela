# Urmaela
Desarrollo proyecto fin de grado superior

Ruta Proyecto

1 ] M5STACK  
2 ] NB IOT
3 ] Raspberry pi 5
4 ] Node Red & Hive MQ
5] THINGSBOARD

Componentes Hardware & Software :

*HARDWARE*
Kit M5 Stack básico
Raspberry pi 5 
Sensor de Temperatura 
Sensor de Ph 
*SOFTWARE*
Node-red
HiveMq
Things Board
 
*----------------------------------------[1] M5STACK]--------------------------------------------------------------*


1.4) Inicio TUTORIAL 

1.0 Información básica de los pasos realizados para realizar el código del proyecto.

1.1 M5STACK HARDWARE.

En este documento explicaremos todo lo que tiene que ver con el M5STACK con nuestro proyecto ( módulos, programación, pruebas… ).
El M5STACK está compuesto por módulos que se acoplan a su estructura principal, esto es algo importante ya que tenemos que tener en cuenta que para realizar algunas acciones extra nuestra M5STACK deberá tener algún módulo especial.
Nosotros, por ejemplo, para utilizar unas librerías específicas para la comunicación de una tarjeta SIM con nuestra M5STACK tuvimos que utilizar un módulo NB-IoT.

1.2 SOFTWARE M5STACK Y SENSORES.

Nuestra forma de hacer el código entero del proyecto fue realizando un código funcional para cada sensor, un código para mandar a dormir el M5STACK, un código para la conexión wifi y un código para el PubSubClient.
El primer código que hicimos fue el de wifi, este código consiste en realizar una conexión entre nuestro dispositivo y la red wifi, una vez realizada la conexión wifi el código hace que el M5STACK dotado con wifi se conecte a la web de HiveMQ iniciando sesion con un usuario. 

Una vez nos llegaron los sensores, comenzamos realizando el código del sensor de humedad, este sensor consiste en dos patas metálicas que se colocan bajo tierra para calcular la humedad del terreno. No utiliza ninguna librería y no resulta difícil encontrar en internet información de este tipo de sensor.

El próximo sensor que pusimos en marcha fue el de temperatura y humedad del aire, este sensor si que utiliza una librería llamada <DHT.h> y es un poco más complicado de encontrar información en caso de utilizar un M5STACK. Tiene una programación sencilla utilizando la librería.

Teniendo los 2 sensores funcionales decidimos comenzar a programar el codigo para mandar a dormir (sleep) el M5STACK, fue sencillo ya que en el propio arduino tras instalar librerias de M5STACK tendras un codigo de ejemplo que precisamente realiza la función de mandar a dormir el dispositivo y volver a encender tras pulsar un botón, nosotros tuvimos que cambiar el código para que el propio M5STACK vuelva a encender sin tocar botones pasado un tiempo x. 


Estando en este punto en lo que realizamos el código del sensor de ph decidimos unir todos los códigos que ya teníamos en uno solo y comprobar si funcionaba todo junto. 
Todo funcionó correctamente, con el código que realizamos el M5STACK encendia solo, calibraba sus sensores, se conectaba a internet y posteriormente tras realizar una lectura de sensores mandaba la información a HiveMQ y se va a dormir, pasados 30 segundos vuelve a encenderse y a realizar el mismo proceso.

El código del sensor de pH que utilizamos fue el código de ejemplo que nos daba el fabricante para arduino, lo único que hicimos fue adaptarlo a nuestro M5STACK.


2.0 Todos los códigos realizados para el proyecto.

2.1 CÓDIGO CONECTARSE A INTERNET Y A HIVEMQ.

Este código nos permite poner nuestros datos para conectarnos a internet y a hiveMQ, el código también realiza el mandar mensajes y el recibir mensajes que se pueden ver en el monitor serie (como mensajes el código mandara números aleatorios de prueba ).

Las librerías que utilizaremos:
#include <WiFi.h>    
#include <PubSubClient.h>   
#include <WiFiClientSecure.h> 
#include <cstring> 

La parte para conectarnos a HiveMQ y wifi:
const char* ssid = "ZYXEL 1";    // nuestro wifi
const char* password = "123456789";  //contraseña wifi
long randomNumber; 
long lastMsg = 0;  
char msg[50];
int value = 0;
int aleatorio = 0;




const char* mqtt_server = "900b3f397cb946f3906cd7107bd93e1a.s2.eu.hivemq.cloud";
const int mqtt_port = 8883; // nuestra ip de server.
const char* mqtt_username = "urmaela"; // name de usuario 
const char* mqtt_password = "urmaela12";  //contraseña.

Necesidad de un certificado: Por alguna extraña razón en una de las webs donde encontramos información de códigos parecidos tenían puesto en el codigo un certificado para poder usar unos comandos, por alguna razón lo necesitamos añadir en el código para que todo funcione correctamente:

static const char* root_ca PROGMEM = R"EOF(
-----BEGIN CERTIFICATE-----
MIIFazCCA1OgAwIBAgIRAIIQz7DSQONZRGPgu2OCiwAwDQYJKoZIhvcNAQELBQAw
TzELMAkGA1UEBhMCVVMxKTAnBgNVBAoTIEludGVybmV0IFNlY3VyaXR5IFJlc2Vh
cmNoIEdyb3VwMRUwEwYDVQQDEwxJU1JHIFJvb3QgWDEwHhcNMTUwNjA0MTEwNDM4
WhcNMzUwNjA0MTEwNDM4WjBPMQswCQYDVQQGEwJVUzEpMCcGA1UEChMgSW50ZXJu
ZXQgU2VjdXJpdHkgUmVzZWFyY2ggR3JvdXAxFTATBgNVBAMTDElTUkcgUm9vdCBY
MTCCAiIwDQYJKoZIhvcNAQEBBQADggIPADCCAgoCggIBAK3oJHP0FDfzm54rVygc
h77ct984kIxuPOZXoHj3dcKi/vVqbvYATyjb3miGbESTtrFj/RQSa78f0uoxmyF+
0TM8ukj13Xnfs7j/EvEhmkvBioZxaUpmZmyPfjxwv60pIgbz5MDmgK7iS4+3mX6U
A5/TR5d8mUgjU+g4rk8Kb4Mu0UlXjIB0ttov0DiNewNwIRt18jA8+o+u3dpjq+sW
T8KOEUt+zwvo/7V3LvSye0rgTBIlDHCNAymg4VMk7BPZ7hm/ELNKjD+Jo2FR3qyH
B5T0Y3HsLuJvW5iB4YlcNHlsdu87kGJ55tukmi8mxdAQ4Q7e2RCOFvu396j3x+UC
B5iPNgiV5+I3lg02dZ77DnKxHZu8A/lJBdiB3QW0KtZB6awBdpUKD9jf1b0SHzUv
KBds0pjBqAlkd25HN7rOrFleaJ1/ctaJxQZBKT5ZPt0m9STJEadao0xAH0ahmbWn
OlFuhjuefXKnEgV4We0+UXgVCwOPjdAvBbI+e0ocS3MFEvzG6uBQE3xDk3SzynTn
jh8BCNAw1FtxNrQHusEwMFxIt4I7mKZ9YIqioymCzLq9gwQbooMDQaHWBfEbwrbw
qHyGO0aoSCqI3Haadr8faqU9GY/rOPNk3sgrDQoo//fb4hVC1CLQJ13hef4Y53CI
rU7m2Ys6xt0nUW7/vGT1M0NPAgMBAAGjQjBAMA4GA1UdDwEB/wQEAwIBBjAPBgNV
HRMBAf8EBTADAQH/MB0GA1UdDgQWBBR5tFnme7bl5AFzgAiIyBpY9umbbjANBgkq
hkiG9w0BAQsFAAOCAgEAVR9YqbyyqFDQDLHYGmkgJykIrGF1XIpu+ILlaS/V9lZL
ubhzEFnTIZd+50xx+7LSYK05qAvqFyFWhfFQDlnrzuBZ6brJFe+GnY+EgPbk6ZGQ
3BebYhtF8GaV0nxvwuo77x/Py9auJ/GpsMiu/X1+mvoiBOv/2X/qkSsisRcOj/KK
NFtY2PwByVS5uCbMiogziUwthDyC3+6WVwW6LLv3xLfHTjuCvjHIInNzktHCgKQ5
ORAzI4JMPJ+GslWYHb4phowim57iaztXOoJwTdwJx4nLCgdNbOhdjsnvzqvHu7Ur
TkXWStAmzOVyyghqpZXjFaH3pO3JLF+l+/+sKAIuvtd7u+Nxe5AW0wdeRlN8NwdC
jNPElpzVmbUq4JUagEiuTDkHzsxHpFKVK7q4+63SM1N95R1NbdWhscdCb+ZAJzVc
oyi3B43njTOQ5yOf+1CceWxG1bQVs5ZufpsMljq4Ui0/1lvh+wjChP4kqKOJ2qxq
4RgqsahDYVvTH9w7jXbyLeiNdd8XM2w9U/t7y0Ff/9yi0GE44Za4rF2LN9d11TPA
mRGunUHBcnWEvgJBQl9nJEiU0Zsnvgc/ubhPgXRR4Xq37Z0j4r7g1SgEEzwxA57d
emyPxgcYxn/eR44/KJ4EBs+lVDR3veyJm+kXQ99b21/+jh5Xos1AnX5iItreGCc=
-----END CERTIFICATE-----
)EOF";

La siguiente parte son básicamente funciones que hemos añadido para que realice las tareas que queremos:

WiFiClientSecure espClient;
PubSubClient client(espClient);


void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}


void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Mensaje recibido en el canal: ");
  Serial.println(topic);
  String content = "";
    for (size_t i = 0; i < length; i++)
    {
        content.concat((char)payload[i]);
    }
    Serial.print(content);
    Serial.println();
}
void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    if (client.connect("ESP32Client", mqtt_username, mqtt_password)) {
      Serial.println("connected");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 2 seconds");
      delay(2000);
    }
  }
}
void setup() {
  Serial.begin(9600);
  setup_wifi();
  espClient.setCACert(root_ca);
  client.setServer(mqtt_server, mqtt_port);
  client.subscribe("MEZUA");
  client.setCallback(callback);
}
void loop() {


   randomNumber = random(1,100);
 if(aleatorio == 0){
 client.publish("M5STACK", msg);
 sprintf(msg, "%ld", randomNumber);  


aleatorio ++;
 }
if(aleatorio >= 1){
  Serial.println(msg);
   aleatorio = 0;
}
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
 // client.publish("M5STACK", "MENSAJE DE PRUEBA");
    client.subscribe("MEZUA");
delay(7000);
}


Aquí termina el código para la conexión wifi y de HiveMQ, aquí queremos agregar una página web que ha sido vital para lograr realizar el código:
( Más abajo en la bibliografía encontramos también este enlace ). 

https://www.luisllamas.es/enviar-y-recibir-mensajes-por-mqtt-con-arduino-y-la-libreria-pubsubclient/


2.2 CÓDIGO SENSOR DE HUMEDAD.

El código que vamos a ver a continuación es el que hace funcionar el sensor de humedad, este sensor estará conectado en el pin 36 de nuestro M5STACK y no precisará de ninguna librería para él, solo usaremos la librería del M5STACK.

Para empezar el código añadimos la librería del M5STACK y añadimos el pin del sensor:

#include <M5Stack.h>




const int humsueloPin = 36;  // Puedes cambiarlo según la conexión física y los pines disponibles
int valHumsuelo;



Seguiremos añadiendo el Void Setup:

void setup() {
  M5.begin();
  Serial.begin(115200);
  pinMode(humsueloPin, INPUT);




  M5.Lcd.begin();
  M5.Lcd.fillScreen(BLACK);
}

Y terminaremos con el Void Loop, aquí, en este apartado tenemos una línea en especial que quizás tengas que cambiar dependiendo del sensor que estés utilizando, es linea funciona añadiendo en orden el valor analogico minimo del sensor, el valor analogico maximo del sensor, el número mínimo que mostrar, el número máximo que puede mostrar:

void loop() {
  // Leer valor del sensor
  int sensorValue = analogRead(humsueloPin);




  // Convertir valor en porcentaje (ajustar según la calibración del sensor)
  valHumsuelo = map(sensorValue, 0, 4075, 0, 100);  // Ajusta estos valores según tu calibración




  // Mostrar valor en la pantalla del M5STACK
  M5.Lcd.setTextSize(2);
  M5.Lcd.setTextColor(WHITE);
  M5.Lcd.setCursor(10, 10);
  M5.Lcd.printf("Humedad del suelo: %d %%", valHumsuelo);




  // Imprimir valor en el Monitor Serial (opcional)
  Serial.print("Valor del sensor: ");
  Serial.println(sensorValue);
  Serial.print("Humedad del suelo: ");
  Serial.print(valHumsuelo);
  Serial.println(" %");


  delay(1000);  // Espera 1 segundo antes de la próxima lectura
}
Para obtener información para realizar el código utilizamos Chat GPT, una herramienta muy buena a la hora de crear códigos pero recomendamos que reviséis el código que proporciona ya que no siempre funciona.


2.3 SENSOR DE TEMPERATURA Y HUMEDAD DEL AIRE.

Con este sensor hemos tenido un pequeño problema, la cosa es que este sensor llamado DHT11 al menos en nuestro caso las primeras 20-40 lecturas no las hace de forma correcta ya que necesita unas lecturas para calibrarse, por esa razón, en el código final hemos añadido un bucle para que este sensor recoja datos x veces en un lapso muy corto de tiempo para comenzar el codigo calibrando. 

Otra cosa a tener en cuenta el DHT11 es que necesita llevar su propia librería, esto será para poder usar después una serie de comandos especiales predefinidos para realizar la lectura del sensor. 

Podemos descargar la libreria aqui:

https://www.arduino.cc/reference/en/libraries/dht-sensor-library/

Comenzaremos el código añadiendo precisamente las librerías del DHT11 y del M5STACK:
#include <M5Stack.h>
#include <DHT.h>

Continuaremos el código añadiendo una línea en la que asignaremos el pin de nuestro sensor y el tipo de sensor del que se trata ( DHT11 ):
#define DHT_PIN 22  // Conecta el pin de datos del DHT11 a este pin
DHT dht(DHT_PIN, DHT11);

Una vez tengamos lo anterior preparado comenzaremos con el void setup:
void setup() {
  M5.begin();
  Serial.begin(115200);
  dht.begin();
  M5.Lcd.begin();
  M5.Lcd.fillScreen(BLACK);
}

Terminaremos añadiendo el void loop al codigo utilizando el dht.read para leer los datos de humedad y temperatura con nuestro sensor:


void loop() {
  // Leer la temperatura y la humedad
  float temperatura = dht.readTemperature();
  float humedad = dht.readHumidity();


  // Mostrar los datos en la pantalla del M5STACK
  M5.Lcd.setTextSize(2);
  M5.Lcd.setTextColor(WHITE);
  M5.Lcd.setCursor(10, 10);
  M5.Lcd.printf("Temperatura: %.1f C", temperatura);
  M5.Lcd.setCursor(10, 40);
  M5.Lcd.printf("Humedad: %.1f %%", humedad);
  // Imprimir los datos en el Monitor Serial (opcional)
  Serial.print("Temperatura: ");
  Serial.print(temperatura);
  Serial.println(" C");
  Serial.print("Humedad: ");
  Serial.print(humedad);
  Serial.println(" %");
delay(1200);
}


2.4 CÓDIGO ENCENDER Y APAGAR M5STACK

El código de sleep del M5STACK que vamos a poner aquí será el código de ejemplo que nos trae el M5STACK en arduino es una versión muy simple y fácil de configurar a nuestro gusto.
Este código nosotros lo hemos cambiado para que despierte sin tener que pulsar el botón, esto es para que cuando quede instalado intente ahorrar la máxima batería posible entrando en periodos de dormir de incluso 1 hora. ( el código que vemos abajo es el que esta sin editar por nosotros ).

/*
*******************************************************************************
*Copyright (c) 2023 by M5Stack
*                  Equipped with M5Core sample source code
*                         M5Core 
* Visit for more information: https://docs.m5stack.com/en/core/gray
* : https://docs.m5stack.com/zh_CN/core/gray
*
* Describe: Power example.  
* Date: 2021/7/21
*******************************************************************************
*/
#include <M5Stack.h>


/* After M5Core is started or reset
the program in the setUp () function will be run, and this part will only be run
once. M5Core

void setup() {
    M5.begin();        // Init M5Core.   M5Core
    M5.Power.begin();  // Init Power module. 
    M5.Power.setWakeupButton(BUTTON_A_PIN);  // Set the screen wake-up button to
                                             // A. 
    M5.Lcd.setBrightness(
        200);  // Set the screen brightness to 200 nits.  
    M5.lcd.setTextSize(2);  // Set the text size to 2. 
}


/* After the program in setup() runs, it runs the program in loop()
  The loop() function is an infinite loop in which the program runs repeatedly
  在setup()loop()
  loop()
void loop() {
    M5.update();  // Read the press state of the key.  读取按键 A, B, C 的状态
    bool c =
        M5.Power
            .isResetbyPowerSW();  // Determine if M5Core is started when powered
                                  // on.  确定M5Core是否在接通电源时启动
    bool d =
        M5.Power.isResetbyDeepsleep();  // Determine if M5Core starts after deep
                                        // sleep. 确定M5Core是否在深度睡眠后启动


    M5.Lcd.println(
        "<Sleep test>");  // The screen prints the formatted string and wraps
                          // the line.  输出格式化字符串并换行
    M5.Lcd.printf("power-on triggered at:%s%s\n\n", (c) ? ("POWER-SW") : (""),
                  (d) ? ("DeepSleep-end") : (""));


    M5.Lcd.printf("Go lightSleep (5s or press buttonA wake up)\n");
    delay(5000);  // delay 5000ms.  延迟5000ms
    /*Restart after 10 seconds of light sleep and continue from the next line
    Calling this function power button will disable the power button to restore
    Please call M5.Power.setPowerBoostKeepOn(false)*/
    5.Power.setPowerBoostKeepOn(false)*/
    M5.Power.lightSleep(SLEEP_SEC(10));
    M5.Lcd.printf("Go lightSleep (press buttonA wake up)\n");
    delay(5000);
    M5.Power.lightSleep(0);


    M5.Lcd.printf("resume.\n\nGo deepSleep (press buttonA wake up) ");
    delay(5000);
    /*After waking up from deep sleep for 0 seconds, the CPU will restart and
    the program will be executed from the beginning
    Calling this function will disable the power button to restore the power
    button Please call M5.Power.setPowerBoostKeepOn(false)*/
   M5.Power.setPowerBoostKeepOn(false)*/
    M5.Power.deepSleep(0);
}


2.5 PRIMER CÓDIGO GRANDE FUNCIONAL

Este código fue el primer código que hicimos usando más de un sensor, usando la función de sleep con el M5STACK y usando la función de mandar la información a nuestro servidor de HiveMQ.

Es importante tener en cuenta que este código necesita que instalemos primero las librerías que necesita, en este caso dejaremos los links de descarga más abajo.
  
Vamos a empezar este código instalando las librerías previamente mencionadas:
#include <WiFi.h>    
#include <PubSubClient.h>  
#include <WiFiClientSecure.h>
 #include <cstring>  
#include <M5Stack.h>
#include <DHT.h>

Lo próximo en el código será añadir la red wifi y algunas funciones que utilizaremos después:
const char* ssid = "iPhone 13 Bry";    // nuestro wifi
const char* password = "laparra12";  //contraseña wifi
long lastMsg = 0;  
char msg[50];
char tmp[50];
char hmd[50];
char pha[50];
int value = 0;
#define DHT_PIN 22  // pin en el que se conecta el sensor de temperatura.
DHT dht(DHT_PIN, DHT11);
const int humsueloPin = 36;  // Puedes cambiarlo según la conexión física y los pines disponibles
int valHumsuelo;
float temperatura;
float humedad;

Siguiendo con el código ahora toca la parte de declarar la pertinente información para el servidor HiveMQ y la certificación para la utilización:
const char* mqtt_server = "900b3f397cb946f3906cd7107bd93e1a.s2.eu.hivemq.cloud";
const int mqtt_port = 8883; // nuestra ip de server.
const char* mqtt_username = "ASDFG"; // name de usuario
const char* mqtt_password = "1234567890'Aa";  //contraseña.




static const char* root_ca PROGMEM = R"EOF(
-----BEGIN CERTIFICATE-----
MIIFazCCA1OgAwIBAgIRAIIQz7DSQONZRGPgu2OCiwAwDQYJKoZIhvcNAQELBQAw
TzELMAkGA1UEBhMCVVMxKTAnBgNVBAoTIEludGVybmV0IFNlY3VyaXR5IFJlc2Vh
cmNoIEdyb3VwMRUwEwYDVQQDEwxJU1JHIFJvb3QgWDEwHhcNMTUwNjA0MTEwNDM4
WhcNMzUwNjA0MTEwNDM4WjBPMQswCQYDVQQGEwJVUzEpMCcGA1UEChMgSW50ZXJu
ZXQgU2VjdXJpdHkgUmVzZWFyY2ggR3JvdXAxFTATBgNVBAMTDElTUkcgUm9vdCBY
MTCCAiIwDQYJKoZIhvcNAQEBBQADggIPADCCAgoCggIBAK3oJHP0FDfzm54rVygc
h77ct984kIxuPOZXoHj3dcKi/vVqbvYATyjb3miGbESTtrFj/RQSa78f0uoxmyF+
0TM8ukj13Xnfs7j/EvEhmkvBioZxaUpmZmyPfjxwv60pIgbz5MDmgK7iS4+3mX6U
A5/TR5d8mUgjU+g4rk8Kb4Mu0UlXjIB0ttov0DiNewNwIRt18jA8+o+u3dpjq+sW
T8KOEUt+zwvo/7V3LvSye0rgTBIlDHCNAymg4VMk7BPZ7hm/ELNKjD+Jo2FR3qyH
B5T0Y3HsLuJvW5iB4YlcNHlsdu87kGJ55tukmi8mxdAQ4Q7e2RCOFvu396j3x+UC
B5iPNgiV5+I3lg02dZ77DnKxHZu8A/lJBdiB3QW0KtZB6awBdpUKD9jf1b0SHzUv
KBds0pjBqAlkd25HN7rOrFleaJ1/ctaJxQZBKT5ZPt0m9STJEadao0xAH0ahmbWn
OlFuhjuefXKnEgV4We0+UXgVCwOPjdAvBbI+e0ocS3MFEvzG6uBQE3xDk3SzynTn
jh8BCNAw1FtxNrQHusEwMFxIt4I7mKZ9YIqioymCzLq9gwQbooMDQaHWBfEbwrbw
qHyGO0aoSCqI3Haadr8faqU9GY/rOPNk3sgrDQoo//fb4hVC1CLQJ13hef4Y53CI
rU7m2Ys6xt0nUW7/vGT1M0NPAgMBAAGjQjBAMA4GA1UdDwEB/wQEAwIBBjAPBgNV
HRMBAf8EBTADAQH/MB0GA1UdDgQWBBR5tFnme7bl5AFzgAiIyBpY9umbbjANBgkq
hkiG9w0BAQsFAAOCAgEAVR9YqbyyqFDQDLHYGmkgJykIrGF1XIpu+ILlaS/V9lZL
ubhzEFnTIZd+50xx+7LSYK05qAvqFyFWhfFQDlnrzuBZ6brJFe+GnY+EgPbk6ZGQ
3BebYhtF8GaV0nxvwuo77x/Py9auJ/GpsMiu/X1+mvoiBOv/2X/qkSsisRcOj/KK
NFtY2PwByVS5uCbMiogziUwthDyC3+6WVwW6LLv3xLfHTjuCvjHIInNzktHCgKQ5
ORAzI4JMPJ+GslWYHb4phowim57iaztXOoJwTdwJx4nLCgdNbOhdjsnvzqvHu7Ur
TkXWStAmzOVyyghqpZXjFaH3pO3JLF+l+/+sKAIuvtd7u+Nxe5AW0wdeRlN8NwdC
jNPElpzVmbUq4JUagEiuTDkHzsxHpFKVK7q4+63SM1N95R1NbdWhscdCb+ZAJzVc
oyi3B43njTOQ5yOf+1CceWxG1bQVs5ZufpsMljq4Ui0/1lvh+wjChP4kqKOJ2qxq
4RgqsahDYVvTH9w7jXbyLeiNdd8XM2w9U/t7y0Ff/9yi0GE44Za4rF2LN9d11TPA
mRGunUHBcnWEvgJBQl9nJEiU0Zsnvgc/ubhPgXRR4Xq37Z0j4r7g1SgEEzwxA57d
emyPxgcYxn/eR44/KJ4EBs+lVDR3veyJm+kXQ99b21/+jh5Xos1AnX5iItreGCc=
-----END CERTIFICATE-----
)EOF";

Después de esto tenemos que añadir el bucle de lectura de los sensores para realizar una calibración de los sensores, este bucle es para el sensor de temperatura y humedad del aire ya que el sensor DHT11 en nuestro caso al menos tarda unas cuantas lecturas en calibrar la temperatura:
WiFiClientSecure espClient;
PubSubClient client(espClient);


bool sensorCalibrado = false;
const int numLecturasCalibracion = 65;
float sumaTemperaturas = 0;
float sumaHumedad = 0;
void calibrarSensor() {  
Serial.println("Calibrando sensores...");
for (int i = 0; i < numLecturasCalibracion; ++i) {
sumaTemperaturas += dht.readTemperature();
sumaHumedad = dht.readHumidity();
delay(500); // Espera entre lecturas
}
  temperatura = sumaTemperaturas / numLecturasCalibracion;
  humedad = sumaHumedad / numLecturasCalibracion;
  Serial.print("Temperatura calibrada: ");
  Serial.println(temperatura);
  sensorCalibrado = true;
}

Ahora vamos a empezar a añadir diferentes funciones para realizar acciones como la de conectarse a nuestro wifi o realizar un callback:
void setup_wifi() {
delay(10);
Serial.println();
Serial.print("Connecting to ");
Serial.println(ssid);
WiFi.begin(ssid, password);
while (WiFi.status() != WL_CONNECTED) {
delay(500);
Serial.print(".");
}
Serial.println("");
Serial.println("WiFi connected");
Serial.println("IP address: ");
Serial.println(WiFi.localIP());
}
void callback(char* topic, byte* payload, unsigned int length) {
Serial.print("Mensaje recibido en el canal: ");
Serial.println(topic);
String content = "";
for (size_t i = 0; i < length; i++) {
content.concat((char)payload[i]);
}
Serial.print(content);
Serial.println();
}
void reconnect() {
while (!client.connected()) {
Serial.print("Attempting MQTT connection...");
if(client.connect("ESP32Client", mqtt_username, mqtt_password)) {
Serial.println("connected");
     
client.publish("HUMEDAD", msg);
sprintf(msg, "%ld", valHumsuelo);  


dtostrf(temperatura, 4, 1, tmp); // Convierte la temperatura a una cadena con un decimal
client.publish("TEMPERATURA", tmp); // Publica la temperatura en el tema "TEMPERATURA"


dtostrf(humedad, 4, 1, hmd);
client.publish("HUMEDAD_AIRE", hmd);


} else {
Serial.print("failed, rc=");
Serial.print(client.state());
Serial.println(" try again in 2 seconds");
delay(2000);
}
}
}

Llegados a este punto ya vamos a comenzar configurando nuestro void setup:
void setup() {
float temperatura = dht.readTemperature();
float humedad = dht.readHumidity();
M5.begin();
pinMode(humsueloPin, INPUT);
dht.begin();
M5.Lcd.begin();
M5.Lcd.fillScreen(BLACK);
Serial.begin(115200);
setup_wifi();
espClient.setCACert(root_ca);
client.setServer(mqtt_server, mqtt_port);
client.subscribe("MEZUA");
client.setCallback(callback);
}

Ahora vamos a finalizar el codigo con el void loop:
void loop() {
if (!sensorCalibrado) {
calibrarSensor();
}
else {
int sensorValue = analogRead(humsueloPin);
float temperatura = dht.readTemperature();
float humedad = dht.readHumidity();


valHumsuelo = map(sensorValue, 0, 4096, 0, 100);  // Ajusta estos valores según tu calibración
Serial.print("Valor del sensor: ");
Serial.println(sensorValue);
Serial.print("Humedad del suelo: ");
Serial.print(valHumsuelo);
Serial.println(" %");
delay(1000);  
client.publish("HUMEDAD", msg);
sprintf(msg, "%ld", valHumsuelo);  
dtostrf(temperatura, 4, 1, tmp); // Convierte la temperatura a una cadena con un decimal
client.publish("TEMPERATURA", tmp); // Publica la temperatura en el tema "TEMPERATURA"
dtostrf(humedad, 4, 1, hmd);
client.publish("HUMEDAD_AIRE", hmd);
if (!client.connected()) {
reconnect();
}
client.loop();
client.subscribe("MEZUA");
delay(5000);
M5.Power.lightSleep(SLEEP_SEC(10));
M5.Power.reset();
}
}

Con todos estos códigos ya juntos y funcionando lo único que nos faltaria es terminar de configurar los 2 sensores que tenemos y realizar el código final con todos los sensores funcionando.


3.0 Librerías que vamos a necesitar.

Las librerías son un elemento que nos permite utilizar comandos adicionales a los “tradicionales” para poder configurar diferentes aparatos como pueden ser unos sensores ( como por ejemplo con el DHT11 ) o para poder usar funciones de escritura nuevas con comandos específicos nuevos ( como por ejemplo con el PubSubClient ):

3.1 CÓMO INSTALAR LIBRERIAS.

Es importante a la hora de programar saber como instalar las librerías ya que las usaremos constantemente.
Las dos formas más comunes de instalar librerias serían las siguientes:
Podemos instalar las librerías desde una página web ( normalmente usaremos reddit ) o también podemos descargar las librerías directamente en el arduino IDE.
Para poder instalarlas desde internet es muy sencillo, buscaremos la librería que queremos instalar y la descargamos en formato .ZIP, posteriormente tendremos que acceder a nuestro arduino IDE, ir al apartado de sketch, darle a incluir biblioteca y posteriormente a añadir biblioteca .ZIP.



La otra forma para poder instalar las librerías sería directamente desde arduino IDE, el problema con esto es que muchas de las librerías que utilizaremos no se encuentran para descargar de forma nativa, por lo tanto, la única forma de poder instalar las librerías será directamente desde internet.
Para descargar la librería desde arduino IDE lo único que tenemos que hacer es presionar en el icono de las librerías y buscar la que estamos buscando.


3.2 LINKS DESCARGA LIBRERIAS UTILIZADAS ( 2.5 )

Aquí vamos a proporcionar todas las librerías utilizadas en el código 2.5 de este documento, con todas estas librerías posteriormente podremos utilizar individualmente todos los códigos desde el 2.1 a 2.5 ya que utilizan las mismas librerías:

Librería WiFi.h:
https://www.arduinolibraries.info/libraries/wi-fi
Librería PubSubClient.h:

https://www.arduinolibraries.info/libraries/pub-sub-client

Libreria WiFiClientSecure.h:

https://github.com/hugethor/WiFiClientSecure

Libreria cstring:

Esta librería la descargamos desde el propio arduino IDE.

Librería M5Stack.h:

https://github.com/m5stack/M5Stack

Librería DHT.h:

https://www.arduino.cc/reference/en/libraries/dht-sensor-library/













*--------------------------------------------[2] NB IOT]--------------------------------------------------------*





*FALTA LA TARJETA NANOSIM PARA EMPEZAR A TESTEAR.


*--------------------------------------------[3] RASPBERRY]------------------------------------------------------*


3.2) Inicio TUTORIAL

INSTALACIÓN UBUNTU

Formatear la SD e instalar el S.O con la herramienta nativa de Raspberry, llamada Raspberry Pi IMAGER.




En el primer arranque poniendo las credenciales de login del WIFI, actualizamos Ubuntu y pasamos al siguiente paso.





























-------------------------------------------------------[4] NODE RED & BROKER MQTT &  HIVE MQ]--------------------------------------------------------------------------


Una vez lista la Raspberry , pasamos a instalar una serie de servicios en ella. En primer lugar NODE RED, después instalaremos el Broker MQTT, y por último crearemos e implementaremos HIVE MQ que hará la función de Servidor.

Node Red          (Para visualizar y gestionar esta parte de comunicación)
Broker MQTT    (Es el servidor que gestiona los mensajes) 
Hive MQ             (Plataforma de mensajería basada en el protocolo MQTT)

4.1) Inicio TUTORIAL

Cómo instalar y configurar Node-RED en Ubuntu 20.04


Paso 1: Actualizar el sistema
Antes de comenzar, es una buena idea actualizar su sistema. Abra una terminal y escriba el siguiente comando:

sudo apt update && sudo apt upgrade
Paso 2: Instalar Node.jsa
Node-RED requiere Node.js para funcionar. Escriba el siguiente comando para instalar Node.js:
sudo apt install nodejs
Verifique la versión de Node.js instalada con el comando:
node -v

Paso 3: Instalar Node-RED

Una vez que Node.js está instalado, puede instalar Node-RED usando el comando Node Package Manager (NPM). Escribe el siguiente comando:

sudo npm install -g --unsafe-perm node-red
Este comando instalará Node-RED globalmente y le permitirá acceder a él desde cualquier parte del sistema.



Paso 4: Inicie Node-RED
Para iniciar Node-RED, escriba el comando:
node-red
Este comando iniciará la interfaz web de Node-RED, accesible a través de su navegador en http://localhost:1880.


Después creamos una cuenta en HIVE MQ para crear nuestro servidor del proyecto.


4.2) INSTALACIÓN BROKER MOSQUITO


Es realmente muy sencillo tanto de instalar como de utilizar pero por si todavía no conoces MQTT, haremos un repaso breve de qué es y cómo funciona.
¿Qué es MQTT?
El protocolo MQTT (también conocido como "Mosquitto") es un protocolo de mensajería ligero y eficiente diseñado para la comunicación entre dispositivos en redes con ancho de banda limitado o conexiones inestables.
Protocolo de Comunicación:
MQTT es un protocolo de comunicación que permite que diferentes dispositivos se envíen mensajes entre sí de manera eficiente y rápida.
Modelo Cliente-Servidor:
Utiliza un modelo cliente-servidor donde los dispositivos pueden ser tanto productores como consumidores de mensajes. Hay un componente central llamado "broker MQTT" que facilita la comunicación entre los dispositivos.
Publicar y Suscribir:
La característica principal de MQTT es el modelo de "publicar" y "suscribir". Un dispositivo puede "publicar" mensajes en un "tema" específico, y otros dispositivos pueden suscribirse a ese tema para recibir esos mensajes.
Topicos (Temas):
Los "temas" son como canales de comunicación. Los dispositivos se conectan a un tema específico para enviar o recibir mensajes. Los mensajes se envían a un tema y cualquier dispositivo que esté suscrito a ese tema recibe el mensaje.





Esquema básico de un sistema MQTT (créditos: IBM)








Cómo instalar y configurar Broker  Mosquitto


El Broker MQTT más popular es Mosquitto. Es un Broker MQTT Open Source que fue creado por la Eclipse Foundation  y es distribuido bajo licencia EPL/EDL. Además es compatible con Windows, Linux y Mac, por lo tanto podremos instalarlo en prácticamente cualquier sitio.
Como en nuestro caso lo que queremos es instalar Mosquitto MQTT en nuestra Raspberry Pi, abrimos una consola y primero nos aseguramos de que tenemos todos los paquetes actualizados.
sudo apt update
sudo apt upgrade
Un vez hecho esto, pasamos a instalar Mosquitto
sudo apt-get install mosquitto mosquitto-clients
Aquí se descargan los paquetes y dependencias necesarias y ya lo tendremos instalado. Ahora veamos cómo podemos probar su correcto funcionamiento antes de entrar de lleno con Node-Red.
Cómo probar Mosquitto MQTT


Lo primero que vamos a hacer es abrir dos consolas SSH al mismo tiempo y que estén ambas logeadas en nuestro servidor de Raspberry Pi. Esto es por que necesitamos similar por un lado un dispositivo que publica datos y otro dispositivos que se suscriben a esos datos.
El broker Mosquitto proporciona dos herramientas para eso llamadas mosquitto_pub y mosquitto _sub que usaremos precisamente para eso.
El la primera consola inicializamos un suscriptor, que escuchará los mensajes publicados en un topic:
mosquitto sub -d -h localhost -p 1883 -t "casa/prueba"


Lo parámetro necesarios son muy simples:
-d: Activa los mensaje de depuración (muy útil mientras probamos cosas)
-h localhost: Especifica dónde está el servidor broker. Si no se indica, por defecto es localhost
- 1883: Puerto de conexión. Por defecto es 1883
- t "casa/prueba": El Topic a donde nos suscribimos

El topic es la parte más fundamental y no es necesario darla de alta en ningún sitio. Solamente indicarla. Si no existe, o más bien si nadie publica en ella, lo peor que puede pasar es que el suscriptor nunca recibirá mensajes de nadie.
Es muy recomendable organizar los topics en forma de árbol, especialmente si vas a tener muchos dispositivos ya que de esa forma quedará todo mucho más claro.
Imagínate que pones sensores de temperatura en toda tu casa. Una forma de organizar los topics sería por ejemplo:
casa/salon/temp
casa/habitacion/juan/temp
casa/habitacion/pedro/temp
casa/cocina/temp
casa/bano/temp
Esta forma de estructurar los topics permite por un lado, tener todo muy claro a nivel de estructura, pero por otro lado permite hacer cosas muy interesantes como utilizar comodines o "wildcard".
Por supuesto un suscriptor puede leer los valores publicado en un determinado topic, como por ejemplo, leer la temperatura de la habitación de pedro tal que así:
casa/habitacion/pedro/temp
Pero también podemos leer todas las actualizaciones de temperatura de todas las habitaciones usando "#"
casa/habitacion/#/temp
Cualquier cliente que se suscriba a este topic recibirá los valores de temperatura de todas las habitaciones. Como ves, si organizas bien tu árbol de topics, es mucho más sencillo hacer cosas así.
Ahora volvamos a nuestro ejemplo y en la segunda consola, vamos a crear o "publicar" un mensaje a nuestro topic "casa/prueba" y lo hacemos usando el comando mosquitto _pub:
mosquitto_pub -d -h localhost -p 1883 -t "casa/prueba" -m "Esto es un mensaje de prueba!"
Copy
Los parámetros utilizados son básicamente los mismos que ya hemos visto con mosquitto _sub, salvo que ahora tenemos uno más:
-m "mensaje": Define el mensaje a enviar
Con esto se enviará el mensaje al broker y a su vez lo recibirán todos los dispositivos suscritos al topic, como la primera consola que dejaste abierta antes y lo mostrará tal que así:

Fichero de configuración de Mosquitto
Para comprobar que esta okey, y como casi todo en un entorno Linux, tenemos disponible un fichero de configuración que está localizado en /etc/mosquitto/mosquitto.conf
Ese fichero guarda toda la configuración del servidor Broker Mosquitto y podremos ajustar los parámetros a nuestro gusto. 






4.3)HiveMQ

Es un broker de mensajes que implementa el protocolo MQTT muy usado en aplicaciones de Internet Of Things(IoT). 


HiveMQ nos da dos posibilidades : 
1ª la primera crea un servidor público , y que la información sea pública 
 2ª la otra es que el servidor sea siendo público , pero la información privada , eso lo hacemos creando un usuario y contraseña en la opción “access management” luego  añadimos nuestro usuario en “web client” , nos suscribimos a un topic , el mismo que tiene que estar en código arduino , ya en el node red , en nuestro topic tenemos que ir a seguridad y poner nuestro usuario y contraseña , el mismo que creamos en el access management y web client , con eso ya podemos recibir datos que nos llegue en el M5 STACK y visualizarlo en el nodo red.

4.4)¿Qué es un bróker de mensajería?

Con un bróker de mensajería, la aplicación fuente (productor) envía un mensaje a un proceso del servidor que podrá proporcionar clasificación de datos, ruteo, traducción de mensajes, persistencia y entrega a todos los respectivos destinatarios (consumidores).

El bróker de mensajería generalmente proporciona todo el manejo del estado y el seguimiento de los clientes, de modo que las aplicaciones individuales no requieren asumir esta responsabilidad, y la complejidad de la entrega de mensajes está integrada en el propio bróker de mensajería.


Existen dos formas básicas de comunicación con un bróker de mensajería:

Publicar y suscribir (temas)
Punto a punto (colas)


4.5)Publicar y suscribir mensajes
En un sistema de mensajería de publicación y suscripción, los productores envían mensajes sobre un tema. En este modelo, el productor se conoce como publicador y el consumidor se conoce como suscriptor. Uno o varios publicadores pueden publicar sobre el mismo tema, y muchos suscriptores pueden recibir un mensaje de uno o varios publicadores. 

Los suscriptores se suscriben a temas y todos los mensajes publicados sobre el tema son recibidos por todos los suscriptores sobre el tema. Este modelo proporciona una entrega simple impulsada por el interés en función de los temas a los que se suscriben las aplicaciones.
4.6)Comunicación punto a punto
Las comunicaciones punto a punto en su forma más simple tienen un productor y un consumidor. Este estilo de mensajería generalmente utiliza una cola para almacenar los mensajes enviados por el productor hasta que el consumidor las reciba. 
El productor de mensajes envía el mensaje a la cola; el consumidor de mensajes recupera los mensajes de la cola y envía una confirmación de que se recibió el mensaje. 

4.7)Puertos 1883  y 8880
Este contenedor nos mapea dos puertos, el 1883 que es el estándar de MQTT y el que se va a usar para publicar y suscribirse a 'topics' y el puerto 8080 donde HiveMQ provee de un 'dashboard' donde monitorizar el servidor(El user/password don admin/hivemq).



*--------------------------------------------------------------[5] THINGSBOARD]-----------------------------------------------------------*



5.1) ¿Qué es? 
ThingsBoard es una plataforma de código abierto para el desarrollo de soluciones de Internet de las cosas (IoT). Proporciona herramientas y servicios que permiten la conexión, recopilación, visualización y análisis de datos provenientes de dispositivos IoT. 
5.2) Características
Algunas características clave de Things Board incluyen:
- Gestión de Dispositivos: Permite registrar, rastrear y gestionar dispositivos IoT de manera eficiente.


- Conectividad: Ofrece soporte para varios protocolos de comunicación como MQTT, CoAP, HTTP y otros, facilitando la integración de dispositivos IoT.
Almacenamiento y Visualización de Datos: Almacena datos de dispositivos y proporciona interfaces gráficas para visualizar y analizar estos datos. Puedes crear dashboards personalizados para monitorizar y controlar dispositivos.


- Reglas y Automatización: Permite definir reglas y lógica de negocio para automatizar acciones basadas en eventos o condiciones específicas.


- Seguridad: Incorpora funciones de seguridad, como autenticación de dispositivos, control de acceso y cifrado de datos, para garantizar la protección de la información sensible.
- Escalabilidad: Puede escalar para gestionar grandes flotas de dispositivos IoT y grandes volúmenes de datos.


- APIs y Extensiones: Ofrece APIs que facilitan la integración con otras aplicaciones y servicios. También permite la creación de extensiones para personalizar y ampliar las funcionalidades.


- Licencia de Código Abierto: Things Board se distribuye bajo la licencia de código abierto Apache 2.0, lo que significa que puedes acceder al código fuente, modificarlo y distribuirlo de acuerdo con los términos de esa licencia.
5.3) Inicio TUTORIAL
 B R Y A N  empieza aqui






[6] PROBLEMAS Y SOLUCIONES]

1er problema

Cuando ya tenemos instalado node red y cambiamos de red , no se inicia por estar en un red diferente.
/comando para iniciar node red / 
node-red-start   
	
Solución
Cuando estemos en la nueva red , en la consola comando de ubuntu ponemos el siguiente comando:

 sudo systemctl enable nodered.service

//sudo para iniciar como administrador 
 
El comando en si es que se inicie node red al iniciar o encender el equipo 
sin necesidad de  usar “ node-red-star”

Como dato ,los nodos que pusimos en el nodered, con la configuración  se quedan guardados

2do problema 
Teniendo los nodos creados en nodered y ponerle el topic los datos no llegaban
 
Solución
HiveMQ nos da dos posibilidades , la primera crea un servidor público , y que la información sea pública , la otra es que el servidor sea siendo público , pero la información privada , eso lo hacemos creando un usuario y contraseña en la opción “access management” .
Luego  añadimos nuestro usuario en “web client” , nos suscribimos a un topic , el mismo que tiene que estar en código arduino , ya en el node red , en nuestro topic tenemos que ir a seguridad y poner nuestro usuario y contraseña , el mismo que creamos en el access management y web client , con eso ya podemos recibir datos que nos llegue en el M5 STACK y visualizarlo en el nodo red.


[7] Anexo]



# Esp32-Cam-FTP

PENSABA EN BORRAR ESTE REPOSITORIO PORQUE LA IDEA PRINCIPAL ERA QUE 2 ESP32 PUDIERAN COMUNICARSE Y ENVIARSE FOTOS MUTAMENTE PERO NO SE PUDO LOGRAR ESTO FUE REMPLAZADO POR TCP/IP PERO ME DI CUENTA QUE ESTO PUEDE SER UTIL PARA ALMACENAR ARCHIVOS UNA IMAGEN O TEXTO Y QUE UN CLIENTE FTP PUEDA VERLO EN EL TELEFONO MEDIANTE UNA APLICACION MOVIL

# NUEVO CODIGO TODO FUNCIONA
```c++
#include <WiFi.h>
#include <WiFiClient.h>

//Librerias
#include "ESP32FtpServer.h"

//Credenciales WiFi
const char* ssid = "Wifi Home 2.4G";
const char* password = "S4m4sw3n0s";

//Objeto FtpServer
FtpServer ftpSrv;

void setup()
{
  //Inicia el puerto serial
  Serial.begin(115200);

  WiFi.begin(ssid, password);
  Serial.println("");
  pinMode(19, INPUT_PULLUP); //pullup GPIO2 para SD_MMC el modo, necesitas 1-15kOm una resistencia conectada a GPIO2 y GPIO19

  //Espera hasta conectar
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("Conectado ");
  Serial.println(ssid);
  Serial.print("IP direccion: ");
  Serial.println(WiFi.localIP());

  if (SD_MMC.begin())
  {
    Serial.println("SD abierta!");

    //Inicia el servidor ftp
    ftpSrv.begin("esp32", "esp32"); //Usuario y password puerto por defecto 21
  }
}

void loop()
{
  //Manejador handle
  ftpSrv.handleFTP();
}
```

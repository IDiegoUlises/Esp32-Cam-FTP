# Esp32-Cam-FTP

* Descargar la libreria FTPClient desde administrar bibloteca
### Codigo copiado
```c++
#include "esp_camera.h"
#include "soc/soc.h"           // Disable brownour problems
#include "soc/rtc_cntl_reg.h"  // Disable brownour problems
#include "driver/rtc_io.h"
#include <WiFi.h>
#include <WiFiClient.h>   
#include "ESP32_FTPClient.h"

#include <NTPClient.h> //For request date and time
#include <WiFiUdp.h>
#include "time.h"

char* ftp_server = "SERVER";
char* ftp_user = "USER";
char* ftp_pass = "PASSWORD";
char* ftp_path = "/USER/;

const char* WIFI_SSID = "NETWORK";
const char* WIFI_PASS = "PASSWORD";

WiFiUDP ntpUDP;
NTPClient timeClient(ntpUDP, "pool.ntp.org", (-3600*3), 60000);

ESP32_FTPClient ftp (ftp_server,ftp_user,ftp_pass, 5000, 2);

//falta la linea(codigo)para abrir la conexion ftp
 ftp.ChangeWorkDir(ftp_path);
  ftp.InitFile("Type I");
  
  String nombreArchivo = timeClient.getFullFormattedTimeForFile()+".jpg";
  Serial.println("Subiendo "+nombreArchivo);
  int str_len = nombreArchivo.length() + 1; 
 
  char char_array[str_len];
  nombreArchivo.toCharArray(char_array, str_len);
  
  ftp.NewFile(char_array);
  ftp.WriteData( fb->buf, fb->len );
  ftp.CloseFile();
```

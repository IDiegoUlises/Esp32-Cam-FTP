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

### Codigo experimental
```c++
#include "esp_camera.h"
#include "FS.h"
#include "SD_MMC.h"

//Declaracion de pins para camara MODEL_AI_THINKER
#define PWDN_GPIO_NUM     32
#define RESET_GPIO_NUM    -1
#define XCLK_GPIO_NUM      0
#define SIOD_GPIO_NUM     26
#define SIOC_GPIO_NUM     27
#define Y9_GPIO_NUM       35
#define Y8_GPIO_NUM       34
#define Y7_GPIO_NUM       39
#define Y6_GPIO_NUM       36
#define Y5_GPIO_NUM       21
#define Y4_GPIO_NUM       19
#define Y3_GPIO_NUM       18
#define Y2_GPIO_NUM        5
#define VSYNC_GPIO_NUM    25
#define HREF_GPIO_NUM     23
#define PCLK_GPIO_NUM     22

//Numero de foto
int fotoNum = 1;


#define WIFI_SSID "Wifi Home"
#define WIFI_PASS "S4m4sw3n0s"

char ftp_server[] = "";
char ftp_user[]   = "";
char ftp_pass[]   = "";

char working_dir[]   = ".";


ESP32_FTPClient ftp (ftp_server,ftp_user,ftp_pass, 5000, 2);
void setup()
{
  //Inicia el puerto serial
  Serial.begin(115200);

  //Configuracion de la camara
  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sscb_sda = SIOD_GPIO_NUM;
  config.pin_sscb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = PWDN_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.pixel_format = PIXFORMAT_JPEG;

  //Verifica si es compatible con PSRAM y elige la configuracion adecuada
  if (psramFound())
  {
    config.frame_size = FRAMESIZE_VGA; //Tamaño de la foto, se pueden elegir QVGA|CIF|VGA|SVGA|XGA|SXGA|UXGA, mientras mas pequeña mejor resolucion
    config.jpeg_quality = 10; //10-63 mientras menor el numero mejor calidad
    config.fb_count = 2;
  }

  else
  {
    config.frame_size = FRAMESIZE_VGA; //Tamaño de la foto, se pueden elegir QVGA|CIF|VGA|SVGA|XGA|SXGA|UXGA, mientras mas pequeña mejor resolucion
    config.jpeg_quality = 10; //10-63 mientras menor el numero mejor calidad
    config.fb_count = 2;
  }

  //Inicializar la camera con la configuracion
  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK)
  {
    Serial.println("Camera no inicializada" + err);
    return;
  }

  //SD incializada correctamente
  if (!SD_MMC.begin())
  {
    Serial.println("SD Card Montada incorrectamente");
    return;
  }

  uint8_t cardType = SD_MMC.cardType();

  //En caso que no se detecte una tarjeta sd
  if (cardType == CARD_NONE)
  {
    Serial.println("No SD Card introducida");
    return;
  }

}

void loop()
{
  TomarFoto();
}

void TomarFoto()
{
  camera_fb_t * fb = NULL;

  //Toma la foto
  fb = esp_camera_fb_get();
  if (!fb)
  {
    Serial.println("La camara no pudo realizar la fotografia");
    return;
  }

  //Ruta de la tarjeta sd donde se guardara la foto
  //String ruta = "/foto" + String(fotoNum) + ".jpg";

  /*
    fs::FS &fs = SD_MMC;
    Serial.println("Foto nombre de archivo: " + ruta);

    File archivo = fs.open(ruta, FILE_WRITE);
    if (!archivo)
    {
      Serial.println("Fallo en abrir el archivo para escribir");
    }

    else
    {
      //Escribe en la tarjeta sd la foto
      archivo.write(fb->buf, fb->len);
      Serial.println("Archivo guardado en la ruta: " + ruta);
    }

    //Cierra el archivo
    archivo.close();
    **/


  ftp.OpenConnection();

  ftp.ChangeWorkDir(working_dir);
  ftp.InitFile("Type I");
  ftp.NewFile("nuevoArchivo.jpg");
  ftp.WriteData( fb->buf, fb->len );
  ftp.CloseFile();

  Serial.println("Imagen enviada por ftp");

  esp_camera_fb_return(fb);

  //Incrementa el valor
  fotoNum++;

  //Retardo de 2 segundos
  delay(2000);
}
```

### Codigo experimental 2 no funciona todavia
```c++
#include "esp_camera.h"
#include "FS.h"
#include "SD_MMC.h"
#include "ESP32_FTPClient.h"
#include <WiFi.h>
#include <WiFiClient.h>

//Declaracion de pins para camara MODEL_AI_THINKER
#define PWDN_GPIO_NUM     32
#define RESET_GPIO_NUM    -1
#define XCLK_GPIO_NUM      0
#define SIOD_GPIO_NUM     26
#define SIOC_GPIO_NUM     27
#define Y9_GPIO_NUM       35
#define Y8_GPIO_NUM       34
#define Y7_GPIO_NUM       39
#define Y6_GPIO_NUM       36
#define Y5_GPIO_NUM       21
#define Y4_GPIO_NUM       19
#define Y3_GPIO_NUM       18
#define Y2_GPIO_NUM        5
#define VSYNC_GPIO_NUM    25
#define HREF_GPIO_NUM     23
#define PCLK_GPIO_NUM     22

//Numero de foto
int fotoNum = 1;


#define WIFI_SSID "Wifi Home"
#define WIFI_PASS "S4m4sw3n0s"

char ftp_server[] = "192.168.100.36";
char ftp_user[]   = "pi";
char ftp_pass[]   = "raspberry";

char working_dir[]   = ".";


ESP32_FTPClient ftp (ftp_server, ftp_user, ftp_pass, 5000, 2);
void setup()
{
  //Inicia el puerto serial
  Serial.begin(115200);

  //Se conecta la a red wifi, en caso de no conectarse quedara en un bucle infinito
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  while (WiFi.status() != WL_CONNECTED)
  {
    Serial.print(".");
    delay(500);
  }

  //Imprime la direccion IP
  Serial.print("\nWiFi connected. IP address: ");
  Serial.println(WiFi.localIP());

  //Configuracion de la camara
  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sscb_sda = SIOD_GPIO_NUM;
  config.pin_sscb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = PWDN_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.pixel_format = PIXFORMAT_JPEG;

  //Verifica si es compatible con PSRAM y elige la configuracion adecuada
  if (psramFound())
  {
    config.frame_size = FRAMESIZE_VGA; //Tamaño de la foto, se pueden elegir QVGA|CIF|VGA|SVGA|XGA|SXGA|UXGA, mientras mas pequeña mejor resolucion
    config.jpeg_quality = 10; //10-63 mientras menor el numero mejor calidad
    config.fb_count = 2;
  }

  else
  {
    config.frame_size = FRAMESIZE_VGA; //Tamaño de la foto, se pueden elegir QVGA|CIF|VGA|SVGA|XGA|SXGA|UXGA, mientras mas pequeña mejor resolucion
    config.jpeg_quality = 10; //10-63 mientras menor el numero mejor calidad
    config.fb_count = 2;
  }

  //Inicializar la camera con la configuracion
  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK)
  {
    Serial.println("Camera no inicializada" + err);
    return;
  }

  //SD incializada correctamente
  if (!SD_MMC.begin())
  {
    Serial.println("SD Card Montada incorrectamente");
    return;
  }

  uint8_t cardType = SD_MMC.cardType();

  //En caso que no se detecte una tarjeta sd
  if (cardType == CARD_NONE)
  {
    Serial.println("No SD Card introducida");
    return;
  }

}

void loop()
{
  TomarFoto();
}

void TomarFoto()
{
  camera_fb_t * fb = NULL;

  //Toma la foto
  fb = esp_camera_fb_get();
  if (!fb)
  {
    Serial.println("La camara no pudo realizar la fotografia");
    return;
  }

  //Ruta de la tarjeta sd donde se guardara la foto
  //String ruta = "/foto" + String(fotoNum) + ".jpg";

  /*
    fs::FS &fs = SD_MMC;
    Serial.println("Foto nombre de archivo: " + ruta);

    File archivo = fs.open(ruta, FILE_WRITE);
    if (!archivo)
    {
      Serial.println("Fallo en abrir el archivo para escribir");
    }

    else
    {
      //Escribe en la tarjeta sd la foto
      archivo.write(fb->buf, fb->len);
      Serial.println("Archivo guardado en la ruta: " + ruta);
    }

    //Cierra el archivo
    archivo.close();
    **/


  ftp.OpenConnection();

  ftp.ChangeWorkDir(working_dir);
  ftp.InitFile("Type I");
  ftp.NewFile("nuevoArchivo.jpg");
  ftp.WriteData( fb->buf, fb->len );
  ftp.CloseFile();

  Serial.println("Imagen enviada por ftp");

  esp_camera_fb_return(fb);

  //Incrementa el valor
  fotoNum++;

  //Retardo de 2 segundos
  delay(2000);
}
```

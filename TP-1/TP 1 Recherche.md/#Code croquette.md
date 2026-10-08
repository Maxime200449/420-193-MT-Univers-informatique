Ca doit compter pour un document en Word(.DOCX) ou Powerpoint(.PPTX)

Tous les détaillent sont fournis sur le site. Sert seulement de démo.
====================================================================
#include <WiFi.h>
#include <WebServer.h>
#include <HX711_ADC.h> //to install for the scale
#include <ESP_Mail_Client.h> //to install to send e-mail
#include <NTPClient.h> //to install to get the local time
#include <WiFiUdp.h>


#include <stdio.h>
#include <string.h>
#include <stdlib.h>


#define SMTP_server "smtp.gmail.com"
#define SMTP_Port 465
#define sender_email "email-created-for-esp32@gmail.com" //change with your informations
#define sender_password "****************" //change with your informations
#define Recipient_email "your-mail@gmail.com" //change with your informations
#define Recipient_name ""
SMTPSession smtp;


const char *ssid = "your_ssid"; //change with your informations
const char *password = "your_wifi_password"; //change with your informations
WebServer server(80);


const int led = 2;


int weight;
int minweight =  15;


const int HX711_dout = 4; 
const int HX711_sck = 5; 


HX711_ADC LoadCell(HX711_dout, HX711_sck);
const int calVal_eepromAdress = 0;
unsigned long t = 0;


WiFiUDP ntpUDP;
NTPClient timeClient(ntpUDP, "europe.pool.ntp.org", 7200, 60000); //change 7200 according to your time zone --> e.g. if you're UTC+2, it will be 3600*2


char time1[6] = "6:30";
char time2[6] = "12:30";
char time3[6] = "18:30";


const int mot = 23;


void handleRoot()
{
    String page = "<!DOCTYPE html>"; //we create a web page here to configure the distributor


    page += "<html lang='fr'>";


    page += "<head>";
    page += "    <title>Server ESP32</title>";
    page += "    <meta http-equiv='refresh' content='60' name='viewport' content='width=device-width, initial-scale=1' charset='UTF-8' />";
    page += "    <link rel='stylesheet' href='https://www.w3schools.com/w3css/4/w3.css'>";
    page += "</head>";


    page += "<body>";
    page += "    <div class='w3-card w3-blue w3-padding-small w3-jumbo w3-center'>";
    page += "        <p>Distributor configuration:</p>";
    page += "    </div>";


    page += "    <div class='w3-bar'>";
    page += "        <h4 style='text-align:left;font-size:70px;margin:20px;'>First distribution:</h4>";
    page += "        <h4 style='text-align:center;font-size:90px;'>"; page+= time1; +"</h4>";
    page += "        <a href='/time1sub' class='w3-bar-item w3-button w3-border w3-jumbo' style='width:30%; height:30%; float:left;'>-</a>";"";
    page += "        <a href='/time1add' class='w3-bar-item w3-button w3-border w3-jumbo' style='width:30%; height:30%; float:right;'>+</a>";"</br></div>";


    page += "    <div class='w3-bar'>";
    page += "        <h4 style='text-align:left;font-size:70px;margin:20px;'>Second distribution:</h4>";
    page += "        <h4 style='text-align:center;font-size:90px;'>"; page+= time2; +"</h4>";
    page += "        <a href='/time2sub' class='w3-bar-item w3-button w3-border w3-jumbo' style='width:30%; height:30%; float:left;'>-</a>";"";
    page += "        <a href='/time2add' class='w3-bar-item w3-button w3-border w3-jumbo' style='width:30%; height:30%; float:right;'>+</a>";"</br></div>";


    page += "    <div class='w3-bar'>";
    page += "        <h4 style='text-align:left;font-size:70px;margin:20px;'>Third distribution:</h4>";
    page += "        <h4 style='text-align:center;font-size:90px;'>"; page+= time3; +"</h4>";
    page += "        <a href='/time3sub' class='w3-bar-item w3-button w3-border w3-jumbo' style='width:30%; height:30%; float:left;'>-</a>";"";
    page += "        <a href='/time3add' class='w3-bar-item w3-button w3-border w3-jumbo' style='width:30%; height:30%; float:right;'>+</a>";"</br></div>";
    
    page += "    <div class='w3-bar'>";
    page += "        <h4 style='text-align:left;font-size:70px;margin:20px;'>Kibble mass per feed (in g):</h4>";
    page += "        <h4 style='text-align:center;font-size:80px;'>"; page+= String(minweight); +"</h4>";
    page += "        <a href='/minweightsub' class='w3-bar-item w3-button w3-border w3-jumbo' style='width:30%; height:30%; float:left;'>-</a>";"";
    page += "        <a href='/minweightadd' class='w3-bar-item w3-button w3-border w3-jumbo' style='width:30%; height:30%; float:right;'>+</a>";"</br></div>";


    page += "    <div class='w3-bar'>";
    page += "        <h4 style='text-align:left;font-size:70px;margin:20px;'>Ration supplémentaire :</h4>";
    page += "        <a href='/bonus' class='w3-bar-item w3-button w3-border w3-jumbo' style='width:30%; height:30%;margin-left:35%;;border-radius:50px;'>BONUS !</a>";"</br></div>";
    
    page += "</body>";


    page += "</html>";


    server.setContentLength(page.length());
    server.send(200, "text/html", page);
}


//Defined voids are used to modify variables such as distribution times and distributed mass


void time1addvoid()
{    
    int hours = atoi(strtok((char *)time1, ":"));
    int minutes = atoi(strtok(NULL, ":"));


    minutes += 5;
    
    if (minutes >= 60) {
        hours += minutes / 60;
        minutes %= 60;
    }
    sprintf(time1, "%02d:%02d", hours, minutes);


    Serial.println(time1);


    server.sendHeader("Location","/");
    server.send(303);
    
}


void time1subvoid()
{
    int hours = atoi(strtok((char *)time1, ":"));
    int minutes = atoi(strtok(NULL, ":"));


    minutes -= 5;


    if (minutes < 0) {
    hours -= 1;
    minutes = 55;
    }


    sprintf(time1, "%02d:%02d", hours, minutes);
    Serial.println(time1);


    server.sendHeader("Location","/");
    server.send(303);
}


void time2addvoid()
{    
    int hours = atoi(strtok((char *)time2, ":"));
    int minutes = atoi(strtok(NULL, ":"));


    minutes += 5;


    if (minutes >= 60) {
        hours += minutes / 60;
        minutes %= 60;
    }
    sprintf(time2, "%02d:%02d", hours, minutes);


    Serial.println(time2);


    server.sendHeader("Location","/");
    server.send(303);
    
}


void time2subvoid()
{
    int hours = atoi(strtok((char *)time2, ":"));
    int minutes = atoi(strtok(NULL, ":"));


    minutes -= 5;


    if (minutes < 0) {
    hours -= 1;
    minutes = 55;
    }


    sprintf(time2, "%02d:%02d", hours, minutes);
    Serial.println(time2);


    server.sendHeader("Location","/");
    server.send(303);
}


void time3addvoid()
{    
    int hours = atoi(strtok((char *)time3, ":"));
    int minutes = atoi(strtok(NULL, ":"));


    minutes += 5;


    if (minutes >= 60) {
        hours += minutes / 60;
        minutes %= 60;
    }
    sprintf(time3, "%02d:%02d", hours, minutes);


    Serial.println(time3);


    server.sendHeader("Location","/");
    server.send(303);
    
}


void time3subvoid()
{
    int hours = atoi(strtok((char *)time3, ":"));
    int minutes = atoi(strtok(NULL, ":"));


    minutes -= 5;


    if (minutes < 0) {
    hours -= 1;
    minutes = 55;
    }


    sprintf(time3, "%02d:%02d", hours, minutes);
    Serial.println(time3);


    server.sendHeader("Location","/");
    server.send(303);
}


void minweightaddvoid()
{
    minweight += 1;
    server.sendHeader("Location","/");
    server.send(303);
}


void minweightsubvoid()
{
    minweight -= 1;
    server.sendHeader("Location","/");
    server.send(303);
}


/ / Sometimes our animal friends are well-behaved and we want to please them, so I've created a Bonus void that delivers a few extra kibbles immediately.


void bonusvoid()
{
    digitalWrite(mot, HIGH);
    delay(5000);
    digitalWrite(mot, LOW);


    server.sendHeader("Location","/");
    server.send(303);
}


void handleNotFound()
{
    server.send(404, "text/plain", "404: Not found");
}


//If there's an error, it will call this void
void errordistrib()
{
    ESP_Mail_Session session;
    session.server.host_name = SMTP_server ;
    session.server.port = SMTP_Port;
    session.login.email = sender_email;
    session.login.password = sender_password;
    session.login.user_domain = "";


    SMTP_Message message;
    message.sender.name = "Kibble dispenser - ALERTS";
    message.sender.email = sender_email;
    message.subject = "ERROR - DISTRIBUTION";
    message.addRecipient(Recipient_name,Recipient_email);


    String textMsg = "Unfortunately, the kibble dispenser doesn't seem to have worked!";


    message.text.content = textMsg.c_str();


    message.text.charSet = "us-ascii";


    message.text.transfer_encoding = Content_Transfer_Encoding::enc_7bit;


    if (!smtp.connect(&session))


        return;


    if (!MailClient.sendMail(&smtp, &message))


        Serial.println("Error sending Email");
}


//void which measures the quantity of kibble
void weightmeasurement()
{
    static boolean newDataReady = 0;
    const int serialPrintInterval = 0; 


    // check for new data/start next conversion:
    if (LoadCell.update()) newDataReady = true;


    // get smoothed value from the dataset:
    if (newDataReady) {
        if (millis() > t + serialPrintInterval) {
        weight = LoadCell.getData();
        Serial.print("Load_cell output val: ");
        Serial.println(weight);
        newDataReady = 0;
        t = millis();
        }
    }


    Serial.println("Weight : " + char(weight));
}


void setup()
{


    //Initialization of the motor
    pinMode(mot, OUTPUT);


    //Initialization serial monitor
    Serial.begin(115200); delay(10);
    Serial.println();
    Serial.println("Starting...");
    
    //Initalization LED
    pinMode(led, OUTPUT);
    digitalWrite(led, LOW);


    //Connection attempt
    WiFi.persistent(false);
    WiFi.begin(ssid, password);
    Serial.print("Tentative de connexion...");


    while (WiFi.status() != WL_CONNECTED)
    {
        Serial.print(".");
        delay(100);
    }


    Serial.println("\n");
    Serial.println("Connexion etablie!");
    Serial.print("Adresse IP: ");
    Serial.println(WiFi.localIP());


    //Define voids for each request
    server.on("/", handleRoot);
    server.on("/time1add", time1addvoid);
    server.on("/time1sub", time1subvoid);
    server.on("/time2add", time2addvoid);
    server.on("/time2sub", time2subvoid);
    server.on("/time3add", time3addvoid);
    server.on("/time3sub", time3subvoid);
    server.on("/minweightadd", minweightaddvoid);
    server.on("/minweightsub", minweightsubvoid);
    server.on("/bonus", bonusvoid);
    server.onNotFound(handleNotFound);
    server.begin();


    Serial.println("Active web server!");
    digitalWrite(led, HIGH);


    //Starting the local time function
    
    timeClient.begin();


    //Sending of an e-mail to confirm successful start-up
    ESP_Mail_Session session;
    session.server.host_name = SMTP_server ;
    session.server.port = SMTP_Port;
    session.login.email = sender_email;
    session.login.password = sender_password;
    session.login.user_domain = "";


    SMTP_Message message;
    message.sender.name = "Kibble dispenser - INITALIZATION";
    message.sender.email = sender_email;
    message.subject = "ESP32 Testing Email";
    message.addRecipient(Recipient_name,Recipient_email);


    String textMsg = "The kibble dispenser has been switched on correctly.";


    message.text.content = textMsg.c_str();


    message.text.charSet = "us-ascii";


    message.text.transfer_encoding = Content_Transfer_Encoding::enc_7bit;


    if (!smtp.connect(&session))
        return;


    if (!MailClient.sendMail(&smtp, &message))
        Serial.println("Error sending Email");
}


void loop()
{
    server.handleClient();
    
    timeClient.update();
    String currenttime = timeClient.getFormattedTime();
    currenttime = currenttime.substring(0, 5);


    if(currenttime == String(time1) || currenttime == String(time2) || currenttime == String(time3)){
        float chrono = 0;
        LoadCell.begin();
        float calibrationValue; 
        calibrationValue = -1040.0; 
        unsigned long stabilizingtime = 2000;
        boolean _tare = true; 
        LoadCell.start(stabilizingtime, _tare);
        if (LoadCell.getTareTimeoutFlag()) {
            Serial.println("Timeout, check MCU>HX711 wiring and pin designations");
            while (1);
        }
        else {
            LoadCell.setCalFactor(calibrationValue); // set calibration value (float)
            Serial.println("Startup is complete");
        }
        while(weight <= minweight){
            digitalWrite(mot, HIGH);
            delay(500);
            chrono += 0.5;
            weightmeasurement();
            if(chrono>=20){
                erreurdistrib();
                break;
            }
        }
        digitalWrite(mot, LOW);
        delay(60000);
    }


    
}


Step 9: Try It!
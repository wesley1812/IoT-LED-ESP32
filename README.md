# IoT-LED-ESP32

#include <WiFi.h>

#define ledPin 2
#define ledPinVermelho 12

const char* ssid = "FIAP-IOT";
const char* password = "F!@p25.IOT"; 
WiFiServer server(80);

void setup() { 
  Serial.begin(115200);
  pinMode(ledPin, OUTPUT);
  pinMode(ledPinVermelho, OUTPUT);
  
  digitalWrite(ledPin, LOW);
  digitalWrite(ledPinVermelho, LOW);
  
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500); 
    Serial.print(".");
  }

  server.begin();
  Serial.println("\nWiFi conectado. IP: ");
  Serial.println(WiFi.localIP());
}

void loop() { 
  WiFiClient client = server.available();
  if (!client) return;

  String request = client.readStringUntil('\r'); 
  client.flush();

  // Lógica de controle
  if (request.indexOf("/LED=LIGADO") != -1) digitalWrite(ledPin, HIGH);
  if (request.indexOf("/LED=DESLIGADO") != -1) digitalWrite(ledPin, LOW);
  if (request.indexOf("/LEDVERMELHO=LIGADO") != -1) digitalWrite(ledPinVermelho, HIGH);
  if (request.indexOf("/LEDVERMELHO=DESLIGADO") != -1) digitalWrite(ledPinVermelho, LOW);

  // Estados atuais
  String statusAzul = digitalRead(ledPin) ? "LIGADO" : "DESLIGADO";
  String statusVermelho = digitalRead(ledPinVermelho) ? "LIGADO" : "DESLIGADO";

  // Resposta HTTP
  client.println("HTTP/1.1 200 OK");
  client.println("Content-Type: text/html");
  client.println("");
  client.println("<!DOCTYPE HTML><html><head>");
  client.println("<meta name=\"viewport\" content=\"width=device-width, initial-scale=1\">");
  client.println("<meta charset=\"UTF-8\">");
  
  // --- INÍCIO DO CSS ---
  client.println("<style>");
  client.println("body { font-family: sans-serif; text-align: center; background-color: #f4f4f9; color: #333; }");
  client.println(".container { max-width: 400px; margin: 20px auto; padding: 20px; background: white; border-radius: 15px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); }");
  client.println("h1 { color: #2c3e50; font-size: 24px; }");
  client.println(".card { margin-bottom: 30px; padding: 10px; border-bottom: 1px solid #eee; }");
  client.println(".status { font-weight: bold; margin: 10px 0; color: #555; }");
  client.println("button { border: none; color: white; padding: 15px 32px; text-align: center; text-decoration: none; display: inline-block; font-size: 16px; margin: 4px 2px; cursor: pointer; border-radius: 8px; transition: 0.3s; width: 80%; }");
  client.println(".btn-on { background-color: #27ae60; }"); // Verde
  client.println(".btn-off { background-color: #c0392b; }"); // Vermelho
  client.println("button:active { transform: scale(0.95); }");
  client.println("</style>");
  // --- FIM DO CSS ---

  client.println("</head><body>");
  client.println("<div class=\"container\">");
  client.println("<h1>Painel de Controle</h1>");
  
  // Seção LED Azul
  client.println("<div class=\"card\">");
  client.print("<h3>LED Azul</h3>");
  client.print("<p class=\"status\">Status: " + statusAzul + "</p>");
  client.println("<a href=\"/LED=LIGADO\"><button class=\"btn-on\">LIGAR</button></a>");
  client.println("<a href=\"/LED=DESLIGADO\"><button class=\"btn-off\">DESLIGAR</button></a>");
  client.println("</div>");

  // Seção LED Vermelho
  client.println("<div class=\"card\">");
  client.print("<h3>LED Vermelho</h3>");
  client.print("<p class=\"status\">Status: " + statusVermelho + "</p>");
  client.println("<a href=\"/LEDVERMELHO=LIGADO\"><button class=\"btn-on\">LIGAR</button></a>");
  client.println("<a href=\"/LEDVERMELHO=DESLIGADO\"><button class=\"btn-off\">DESLIGAR</button></a>");
  client.println("</div>");

  client.println("</div></body></html>");

  delay(1);
}

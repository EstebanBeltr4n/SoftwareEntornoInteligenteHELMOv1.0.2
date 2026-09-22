# Descripción HELMO

---

HELMO es un sistema de monitoreo inteligente de calidad de agua. Nodos de campo (Heltec WiFi LoRa 32, firmware en C++/Arduino) miden pH, turbidez y nivel, y transmiten por LoRaWAN a The Things Stack, donde un decodificador en JavaScript estructura la telemetría. Un backend en Node.js/Express la persiste en MongoDB (local y Atlas) y la visualiza en Grafana, permitiendo vigilancia remota y alertas tempranas en plantas de tratamiento y tanques de agua.

---


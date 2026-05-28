# PilotSPA — Balboa GS Controller for Home Assistant

Fork de [zarzak12/pilotspa](https://github.com/zarzak12/pilotspa), converti pour être compilé avec **PlatformIO** et adapté pour la carte **ESP32-C3**.

## Description

Contrôleur pour spa Balboa équipé du système GS avec afficheur **VL801D**. L'ESP32-C3 se connecte au bus d'affichage du spa pour lire l'état (température, pompes, chauffage…) et envoyer des commandes (boutons virtuels). L'intégration Home Assistant se fait via **MQTT**.

## Différences par rapport au fork d'origine

- Converti de l'IDE Arduino vers **PlatformIO**
- Cible **ESP32-C3** (`esp32-c3-devkitm-1`) au lieu d'un ESP32 standard
- `SPIFFS` remplacé par **LittleFS**
- Librairies mises à jour : `mathieucarbou/ESPAsyncWebServer` + `mathieucarbou/AsyncTCP`
- ElegantOTA supprimé
- GPIO adaptés à l'ESP32-C3 (évite les pins 11–17 réservées au flash SPI interne)
- `IRAM_ATTR` ajouté à la déclaration ET la définition de l'ISR (requis sur ESP32-C3)
- `INPUT_PULLDOWN` sur les pins clock et display pour stabiliser les entrées

## Matériel requis

- ESP32-C3 DevKitM-1
- Spa Balboa avec système GS et afficheur VL801D
- 3 fils vers le connecteur d'affichage du spa

## Câblage

| Signal Balboa | GPIO ESP32-C3 |
|---------------|---------------|
| Clock         | GPIO 4        |
| Display (RX)  | GPIO 5        |
| Button (TX)   | GPIO 6        |

> **Note :** Ne pas utiliser les GPIO 11–17 sur ESP32-C3 — ils sont connectés au flash SPI interne.

## Dépendances

Gérées automatiquement par PlatformIO via `platformio.ini` :

- [dawidchyrzynski/home-assistant-integration](https://github.com/dawidchyrzynski/arduino-home-assistant) `^2.1.0`
- [mathieucarbou/AsyncTCP](https://github.com/mathieucarbou/AsyncTCP) `^3.2.0`
- [mathieucarbou/ESPAsyncWebServer](https://github.com/mathieucarbou/ESPAsyncWebServer) `^3.3.0`

## Installation

### 1. Cloner le dépôt

```bash
git clone https://github.com/<votre-repo>/pilotspa.git
cd pilotspa
```

### 2. Compiler et flasher le firmware

```bash
pio run --target upload
```

### 3. Flasher les fichiers web (LittleFS)

```bash
pio run --target uploadfs
```

### 4. Configuration initiale

Au premier démarrage (ou si l'EEPROM est vierge), l'ESP32 crée un point d'accès Wi-Fi :

- **SSID :** `OpenSpa-config`
- **Mot de passe :** `openspa2801`

Connectez-vous à ce réseau et accédez à `http://192.168.4.1` pour renseigner :

- SSID et mot de passe Wi-Fi
- Adresse IP, identifiants du broker MQTT
- Identifiants de l'interface web

L'ESP32 redémarre automatiquement après la sauvegarde.

## Interface web

Une fois connecté au réseau :

| Page | URL |
|------|-----|
| Tableau de bord | `http://<ip>/` |
| Pilotage | `http://<ip>/pilotage` |
| Configuration | `http://<ip>/config` |

## Intégration Home Assistant

L'appareil se déclare automatiquement via MQTT Discovery. Les entités exposées :

- Températures eau (mesurée et consigne) — thermostat HVAC
- État chauffage, pompes, blower, lumière
- Mode (Standard / Eco / Sleep / Ice)
- Filtration
- Adresse IP, signal Wi-Fi

## Structure du projet

```
pilotspa/
├── src/
│   └── main.cpp                  # Programme principal
├── lib/
│   └── Balboa_GS_Interface/      # Librairie de communication avec le spa
│       ├── Balboa_GS_Interface.h
│       └── Balboa_GS_Interface.cpp
├── data/                         # Fichiers web (LittleFS)
│   ├── index.html
│   ├── pilotage.html
│   ├── config.html
│   ├── w3.css
│   ├── perso.css
│   └── favicon.ico
└── platformio.ini
```

## Licence

Voir le dépôt d'origine : [zarzak12/pilotspa](https://github.com/zarzak12/pilotspa)

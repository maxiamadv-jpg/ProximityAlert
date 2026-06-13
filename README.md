# ProximityAlert — Android + WearOS

App che fa vibrare lo smartwatch quando smartphone e smartwatch si allontanano oltre la soglia configurata (default: 2 metri).

---

## Come funziona

```
Smartphone (BLE scan) ──RSSI──▶ Stima distanza ──▶ Wearable Data Layer ──▶ Smartwatch (vibra)
```

1. **Smartphone** scansiona continuamente il RSSI Bluetooth dei dispositivi accoppiati
2. Converte l'RSSI in distanza stimata con la formula standard: `d = 10^((TxPower - RSSI) / (10 × n))`
3. Se la distanza supera la soglia → invia un messaggio via **Wearable Data Layer API** (Google)
4. **Smartwatch** riceve il messaggio nel `WearListenerService` e attiva la **vibrazione a pattern**
5. Quando i dispositivi si riavvicinano → vibrazione breve di conferma + UI torna normale

---

## Struttura del progetto

```
ProximityAlert/
├── phone/                          ← Modulo smartphone
│   └── src/main/
│       ├── AndroidManifest.xml
│       └── java/com/proximityalert/phone/
│           ├── MainActivity.kt             # UI principale
│           ├── ProximityService.kt         # Foreground Service BLE + distanza
│           └── WearMessageListenerService.kt  # Riceve messaggi dal watch
│
├── wear/                           ← Modulo smartwatch (WearOS)
│   └── src/main/
│       ├── AndroidManifest.xml
│       └── java/com/proximityalert/wear/
│           ├── WearMainActivity.kt         # UI watch (stato + allerta)
│           └── WearListenerService.kt      # Riceve messaggi + vibrazione
│
├── build.gradle
└── settings.gradle
```

---

## Requisiti

| Componente | Minimo |
|---|---|
| Android SDK | API 26 (Android 8.0) |
| WearOS | API 26 |
| Bluetooth | BLE 4.0+ |
| Dipendenze | `play-services-wearable:18.1.0` |

---

## Setup in Android Studio

### 1. Crea il progetto
```
File → New → New Project → "Phone and Tablet" + "Wear OS"
Package: com.proximityalert
```

### 2. Sostituisci i file
Copia tutti i file di questo progetto nelle posizioni corrispondenti.

### 3. Aggiungi le dipendenze mancanti nel `phone/build.gradle`:
```gradle
implementation 'com.google.android.gms:play-services-wearable:18.1.0'
implementation 'com.google.android.material:material:1.11.0'
```

### 4. Aggiungi in `wear/build.gradle`:
```gradle
implementation 'com.google.android.gms:play-services-wearable:18.1.0'
implementation 'androidx.wear:wear:1.3.0'
```

### 5. Stessa Application ID
Verifica che `applicationId` nel modulo phone sia uguale a quello nel modulo wear (stessa base).

---

## Deploy

### Smartwatch reale
1. Abilita **Developer Mode** sull'orologio: Impostazioni → Sistema → Info → tocca 7 volte "Numero Build"
2. Abilita **ADB via Wi-Fi**: Impostazioni → Sviluppatore → Debug ADB
3. In Android Studio: `Run → Run 'wear'` selezionando il watch

### Emulatore
1. AVD Manager → Crea dispositivo → Wear OS
2. Avvia emulatore phone + emulatore wear
3. Collega tramite `adb` o dall'emulatore stesso

---

## Permessi richiesti (smartphone)

| Permesso | Uso |
|---|---|
| `BLUETOOTH_SCAN` | Scansione BLE |
| `BLUETOOTH_CONNECT` | Connessione ai dispositivi |
| `BLUETOOTH_ADVERTISE` | Advertising BLE |
| `ACCESS_FINE_LOCATION` | Richiesto da Android < 12 per BLE |
| `FOREGROUND_SERVICE` | Servizio in background |
| `POST_NOTIFICATIONS` | Notifica foreground |

---

## Note tecniche

### Precisione distanza BLE
La stima RSSI → distanza è **approssimativa** (±0.5-1.5m). Fattori che influenzano:
- Ostacoli fisici (muri, corpi umani)
- Interferenze RF (Wi-Fi, altri BLE)
- Orientamento dei dispositivi

Per aumentare la precisione in ambienti chiusi aumenta `PATH_LOSS_EXP` da 2.5 a 3.0-3.5 in `ProximityService.kt`.

### Consumo batteria
Il servizio usa `SCAN_MODE_LOW_LATENCY` per massima reattività. Per risparmiare batteria usa `SCAN_MODE_BALANCED` ma con risposta più lenta.

---

## Personalizzazione

### Cambiare il pattern di vibrazione (WearListenerService.kt)
```kotlin
// Pattern: delay, vibra, pausa, vibra, pausa...
val pattern = longArrayOf(0, 300, 200, 300, 200, 300)
```

### Cambiare la soglia default (ProximityService.kt)
```kotlin
// Modifica la soglia default da 2.0 a X metri
private var thresholdMeters = 2.0f  // ← cambia qui
```

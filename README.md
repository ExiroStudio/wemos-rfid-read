# Wemos RFID Read

This project demonstrates how to read multiple blocks of data from Mifare RFID cards using the Wemos D1 Mini (ESP8266) and the RC522 RFID module. This solution is useful for access control systems, reading stored information on RFID cards, and other smart applications.

## Features
- Read data from multiple blocks on RFID cards using the RC522 module
- Authenticate and read data from Mifare cards
- Works seamlessly with the Wemos D1 Mini (ESP8266)
- Simple and clear implementation with comments for beginners

## Getting Started

### Requirements
- **Hardware**:
  - Wemos D1 Mini (ESP8266)
  - RC522 RFID Reader/Writer module
  - Mifare RFID cards
  - Jumper wires

- **Software**:
  - Arduino IDE
  - Required libraries: 
    - [MFRC522](https://github.com/miguelbalboa/rfid)
    - ESP8266 board support for Arduino

### Circuit Diagram
Connect the RC522 RFID module to the Wemos D1 Mini as follows:
- **SDA (SS_PIN)** → D1
- **SCK** → D5
- **MOSI** → D7
- **MISO** → D6
- **IRQ** → Not connected
- **GND** → GND
- **RST** → D2
- **VCC** → 3.3V

### Installation

1. Clone this repository:
    ```bash
    git clone https://github.com/ExiroStudio/wemos-rfid-read.git
    ```

2. Install the required libraries in the Arduino IDE:
    - Search and install the **MFRC522** library from the Library Manager.
    - Install **ESP8266** board support in the Board Manager.

3. Open the Arduino IDE and load the provided sketch from this repository.

4. Compile and upload the code to your Wemos D1 Mini.

### How to Use

1. After uploading the sketch, open the Serial Monitor (baud rate: 115200).
2. Place an RFID card near the RC522 module.
3. The system will authenticate, read data from multiple blocks, and display the read data on the Serial Monitor.

## Code Example

Here is the example code that reads data from an RFID card using the Wemos D1 Mini and the RC522 module:

```cpp
#include <SPI.h>
#include <MFRC522.h>

#define RST_PIN D2  // Pin untuk Reset
#define SS_PIN D1   // Pin untuk Slave Select (SDA)

MFRC522 mfrc522(SS_PIN, RST_PIN);  // Buat instance MFRC522

// Default key untuk otentikasi
MFRC522::MIFARE_Key key;

void setup() {
  Serial.begin(115200);  // Inisialisasi komunikasi serial dengan PC
  while (!Serial);       // Tunggu hingga serial port terbuka
  SPI.begin();           // Inisialisasi SPI bus
  mfrc522.PCD_Init();    // Inisialisasi MFRC522
  delay(4);              // Delay opsional, beberapa board membutuhkan waktu lebih lama setelah inisialisasi
  mfrc522.PCD_DumpVersionToSerial();  // Tampilkan detail dari pembaca kartu MFRC522
  Serial.println(F("Scan PICC to read data..."));

  // Set default key
  for (byte i = 0; i < 6; i++) {
    key.keyByte[i] = 0xFF;
  }
}

void loop() {
  // Reset loop jika tidak ada kartu baru yang terdeteksi
  if (!mfrc522.PICC_IsNewCardPresent()) {
    return;
  }

  // Pilih salah satu kartu
  if (!mfrc522.PICC_ReadCardSerial()) {
    return;
  }

  // Data buffer untuk menyimpan data yang dibaca
  char data[64];
  byte dataLength = 0;

  // Membaca data dari beberapa blok berturut-turut, mulai dari blok 1
  for (byte i = 0; i < 4; i++) {
    byte block = 1 + i;  // Mulai dari blok 1

    // Otentikasi dengan blok menggunakan kunci A
    MFRC522::StatusCode status = mfrc522.PCD_Authenticate(MFRC522::PICC_CMD_MF_AUTH_KEY_A, block, &key, &(mfrc522.uid));
    if (status != MFRC522::STATUS_OK) {
      Serial.print(F("PCD_Authenticate() failed: "));
      Serial.println(mfrc522.GetStatusCodeName(status));
      return;
    }

    // Baca data dari blok
    byte buffer[18];
    byte size = sizeof(buffer);
    status = mfrc522.MIFARE_Read(block, buffer, &size);
    if (status != MFRC522::STATUS_OK) {
      Serial.print(F("MIFARE_Read() failed: "));
      Serial.println(mfrc522.GetStatusCodeName(status));
      return;
    }

    // Salin data yang dibaca ke data buffer
    for (byte j = 0; j < 16; j++) {
      data[dataLength++] = buffer[j];
    }
  }

  // Tambahkan null terminator pada akhir string
  data[dataLength] = '\0';

  // Cetak data yang dibaca dari kartu RFID
  Serial.print(F("Data read from the card: "));
  Serial.println(data);

  // Berhentikan kartu agar dapat membaca kartu lainnya
  mfrc522.PICC_HaltA();
  mfrc522.PCD_StopCrypto1();
}
```

### Reading Data from RFID Card
This example reads four consecutive blocks from the RFID card starting at block 1. The data is printed to the Serial Monitor. You can adjust the blocks being read based on your application’s needs.

## Contributing

Contributions are welcome! Feel free to fork this repository and submit pull requests to add new features or improve the code.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

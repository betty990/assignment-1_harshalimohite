# assignment-1_harshalimohite
#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>
#define SERVICE_UUID        "00000002-0000-0000-FDFD-FDFDFDFDFDFD"
#define CHARACTERISTIC_UUID_TEMPERATURE "00002A1E-0000-1000-8000-00805F9B34FB"
#define CHARACTERISTIC_UUID_HUMIDITY "00002A6E-0000-1000-8000-00805F9B34FB"
BLECharacteristic *pCharacteristicTemperature;
BLECharacteristic *pCharacteristicHumidity;

void setup() {
    Serial.begin(115200);
    BLEDevice::init("ESP32_BLE");

    BLEServer *pServer = BLEDevice::createServer();
    BLEService *pService = pServer->createService(SERVICE_UUID);

    pCharacteristicTemperature = pService->createCharacteristic(
                                    CHARACTERISTIC_UUID_TEMPERATURE,
                                    BLECharacteristic::PROPERTY_READ |
                                    BLECharacteristic::PROPERTY_NOTIFY
                                  );
    pCharacteristicTemperature->addDescriptor(new BLE2902());

    pCharacteristicHumidity = pService->createCharacteristic(
                                   CHARACTERISTIC_UUID_HUMIDITY,
                                   BLECharacteristic::PROPERTY_READ |
                                   BLECharacteristic::PROPERTY_NOTIFY
                                 );
    pCharacteristicHumidity->addDescriptor(new BLE2902());

    pService->start();
    BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();
    pAdvertising->addServiceUUID(SERVICE_UUID);
    pAdvertising->setScanResponse(true);
    pAdvertising->setMinPreferred(0x06);  // functions that help with iPhone connections issue
    pAdvertising->setMinPreferred(0x12);
    BLEDevice::startAdvertising();
    Serial.println("Waiting a client connection to notify...");
}

void loop() {
    float temperature = 22.0; // Simulated temperature value
    float humidity = 45.0; // Simulated humidity value
    Serial.print("temperature");
    Serial.print(temperature);
    Serial.print(" C, Humidity:");
    Serial.print(humidity);
    Serial.println("%");
    char tempStr[8];
    char humStr[8];
    dtostrf(temperature, 1, 2, tempStr);
    dtostrf(humidity, 1, 2, humStr);

    pCharacteristicTemperature->setValue(tempStr);
    pCharacteristicHumidity->setValue(humStr);

    pCharacteristicTemperature->notify();
    pCharacteristicHumidity->notify();

    delay(2000); // Notify every 2 seconds
}

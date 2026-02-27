---
name: nimble-service-skill
description: Create and edit BLE GATT services with NimBLE. Use when creating, editing, or refactoring BLE services, characteristics, descriptors, or callbacks.
metadata:
  author: Leeor Nahum
  version: "1.3"
---

# NimBLE BLE Service Guide

## Authoritative References

Consult these official documents for UUIDs, format values, unit codes, and specifications:

- **[Bluetooth Assigned Numbers](https://www.bluetooth.com/wp-content/uploads/Files/Specification/HTML/Assigned_Numbers/out/en/Assigned_Numbers.pdf)**
- **[GATT Specification Supplement](https://btprodspecificationrefs.blob.core.windows.net/gatt-specification-supplement/GATT_Specification_Supplement.pdf)**
- **[GATT XML Specification Repository](https://github.com/oesmith/gatt-xml)**

For NimBLE-specific methods, enums, and properties, check the NimBLE library headers (e.g., `NimBLECharacteristic.h`, `NimBLE2904.h`) in the project dependencies.

## UUID Conventions

1. Check Bluetooth Assigned Numbers PDF for an official UUID that fits the use case
2. If an official UUID exists and is appropriate, use the short form (e.g., `"180F"`)
3. If no official UUID fits, generate a custom 128-bit UUID:

```bash
python -c "import uuid; print(str(uuid.uuid4()))"
```

## Service Class Template

UUIDs are declared as `static const` members inside the class. This scopes them to the class and prevents naming collisions across libraries.

Variable names should match the UUID constant prefix. For services and characteristics from Bluetooth Assigned Numbers, use their canonical names (e.g., `SERVICE_UUID` → `sensor_service`, `DATA_CHARACTERISTIC_UUID` → `data_characteristic`).

### Header (.h)

```cpp
#ifndef BLE_SENSOR_SERVICE_H
#define BLE_SENSOR_SERVICE_H

#include <NimBLEDevice.h>

class BLESensorServiceClass {
  public:
    static const NimBLEUUID SERVICE_UUID;

    static const NimBLEUUID DATA_CHARACTERISTIC_UUID;
    static const NimBLEUUID CONTROL_CHARACTERISTIC_UUID;

    bool startService();

    NimBLEService* getService() { return sensor_service; }

    bool setData(uint16_t value, bool notify = false);
    bool isDataSubscribed() { return data_subscribed; }
    NimBLECharacteristic* getDataCharacteristic() { return data_characteristic; }

    NimBLECharacteristic* getControlCharacteristic() { return control_characteristic; }

  private:
    NimBLEService* sensor_service = nullptr;

    NimBLECharacteristic* data_characteristic = nullptr;
    NimBLECharacteristic* control_characteristic = nullptr;

    bool data_subscribed = false;

    void createDataCharacteristic();
    void createControlCharacteristic();

    class DataCallbacks;
    class ControlCallbacks;
};

extern BLESensorServiceClass BLESensorService;

#endif

```

### Implementation (.cpp)

```cpp
#include "ble_sensor_service.h"

const NimBLEUUID BLESensorServiceClass::SERVICE_UUID("...");

const NimBLEUUID BLESensorServiceClass::DATA_CHARACTERISTIC_UUID("...");
const NimBLEUUID BLESensorServiceClass::CONTROL_CHARACTERISTIC_UUID("...");

BLESensorServiceClass BLESensorService;

class BLESensorServiceClass::DataCallbacks : public NimBLECharacteristicCallbacks {
  public:
    DataCallbacks(BLESensorServiceClass* pService) : service(pService) {}

    void onSubscribe(NimBLECharacteristic* pCharacteristic, NimBLEConnInfo& connInfo, uint16_t subValue) override {
      service->data_subscribed = (subValue != 0);

      // Optional: Additional logic
    }

  private:
    BLESensorServiceClass* service;
};

class BLESensorServiceClass::ControlCallbacks : public NimBLECharacteristicCallbacks {
  public:
    void onWrite(NimBLECharacteristic* pCharacteristic, NimBLEConnInfo& connInfo) override {
      NimBLEAttValue value = pCharacteristic->getValue();

      // Process command using value.data() and value.size()
    }
};

bool BLESensorServiceClass::startService() {
  NimBLEServer* pServer = NimBLEDevice::getServer();
  if (pServer == nullptr) return false;

  sensor_service = pServer->getServiceByUUID(SERVICE_UUID);
  if (sensor_service == nullptr) {
    sensor_service = pServer->createService(SERVICE_UUID);
  }

  createDataCharacteristic();
  createControlCharacteristic();

  return sensor_service->start();
}

void BLESensorServiceClass::createDataCharacteristic() {
  if (sensor_service == nullptr) return;

  if (data_characteristic == nullptr) {
    data_characteristic = sensor_service->getCharacteristic(DATA_CHARACTERISTIC_UUID);
    if (data_characteristic == nullptr) {
      data_characteristic = sensor_service->createCharacteristic(
        DATA_CHARACTERISTIC_UUID,
        NIMBLE_PROPERTY::READ | NIMBLE_PROPERTY::NOTIFY
      );

      data_characteristic->setCallbacks(new DataCallbacks(this));

      NimBLEDescriptor* user_description = data_characteristic->createDescriptor(NimBLEUUID("2901"), NIMBLE_PROPERTY::READ);
      user_description->setValue("Sensor Data");

      NimBLE2904* presentation_format = (NimBLE2904*)data_characteristic->createDescriptor(NimBLEUUID("2904"), NIMBLE_PROPERTY::READ);
      presentation_format->setFormat(NimBLE2904::FORMAT_UINT16);
      presentation_format->setExponent(0x00);
      presentation_format->setUnit(0x2700);
      presentation_format->setNamespace(0x00);
      presentation_format->setDescription(0x0000);

      NimBLEDescriptor* valid_range = data_characteristic->createDescriptor(NimBLEUUID("2906"), NIMBLE_PROPERTY::READ);
      uint16_t range[2] = { 0, 1000 };
      valid_range->setValue((uint8_t*)range, sizeof(range));

      uint16_t initial = 0;
      data_characteristic->setValue((uint8_t*)&initial, sizeof(initial));
    }
  }
}

void BLESensorServiceClass::createControlCharacteristic() {
  if (sensor_service == nullptr) return;

  if (control_characteristic == nullptr) {
    control_characteristic = sensor_service->getCharacteristic(CONTROL_CHARACTERISTIC_UUID);
    if (control_characteristic == nullptr) {
      control_characteristic = sensor_service->createCharacteristic(
        CONTROL_CHARACTERISTIC_UUID,
        NIMBLE_PROPERTY::WRITE
      );

      control_characteristic->setCallbacks(new ControlCallbacks());

      NimBLEDescriptor* user_description = control_characteristic->createDescriptor(NimBLEUUID("2901"), NIMBLE_PROPERTY::READ);
      user_description->setValue("Control");

      NimBLE2904* presentation_format = (NimBLE2904*)control_characteristic->createDescriptor(NimBLEUUID("2904"), NIMBLE_PROPERTY::READ);
      presentation_format->setFormat(NimBLE2904::FORMAT_UINT8);
      presentation_format->setExponent(0x00);
      presentation_format->setUnit(0x2700);
      presentation_format->setNamespace(0x00);
      presentation_format->setDescription(0x0000);
    }
  }
}

bool BLESensorServiceClass::setData(uint16_t value, bool notify) {
  if (data_characteristic == nullptr) return false;

  // Optional: Additional logic (validation, transformation, side effects)

  data_characteristic->setValue((uint8_t*)&value, sizeof(value));
  if (notify) {
    data_characteristic->notify();
  }

  return true;
}

```

## Usage Example

```cpp
// In BLE initialization (after NimBLEDevice::createServer() has been called)
BLESensorService.startService();

// Advertise the service UUID if you want clients to discover this service (Optional)
NimBLEAdvertising* pAdvertising = NimBLEDevice::getAdvertising();
pAdvertising->addServiceUUID(BLESensorServiceClass::SERVICE_UUID);

// Example: Update sensor data from anywhere and optionally notify clients
BLESensorService.setData(123, true);
```

## NimBLE Server Singleton

NimBLE uses a singleton pattern for the BLE server - there is only one server per device. This means:

- **No constructor parameters needed** - services don't require a server pointer to be passed in
- **Global access** - the extern singleton pattern lets you call `BLESensorService.setData(...)` from anywhere
- **Simplified initialization** - just call `startService()` after `NimBLEDevice::createServer()` has been called

## Descriptor Conventions

### Namespace/Description Rule

These fields are linked in the 0x2904 descriptor:

- If `Description = 0x0000` → set `Namespace = 0x00`
- If `Description != 0x0000` (Bluetooth SIG enumeration) → set `Namespace = 0x01`

### Common Format/Unit Combinations

| Data Type | Format | Unit |
| ----------- | -------- | ------ |
| Percentage | `FORMAT_UINT8` | `0x27AD` |
| Acceleration (m/s²) | `FORMAT_FLOAT32` | `0x2713` |
| Angular velocity (rad/s) | `FORMAT_FLOAT32` | `0x2763` |
| Temperature (°C) | `FORMAT_SINT16` | `0x272F` |
| Boolean/Unitless | `FORMAT_BOOLEAN` or `FORMAT_UINT8` | `0x2700` |
| String | `FORMAT_UTF8` | `0x2700` |

## Service Ordering

When adding services to the BLE stack, maintain consistent ordering:

- Core/vital services first (Device Information, Error Report)
- Application-specific services in logical groups
- Utility services that rarely change (OTA) last

This ordering should be consistent across the codebase for predictability.

---
title: "Flutter에서 ble사용하기 flutter-reactive-ble"
categories:
  - flutter
tags:
  - flutter
  - ble
  - bluetooth
toc: true
toc-sticky: true
---

bluetooth 통신에는 bluetooth classic과 bluetooth low energy가 있습니다.

[flutter_reactive_ble](https://pub.dev/packages/flutter_reactive_ble) 는 ble(bluetooth low energy) 통신을 지원하는 패키지입니다.

다른 유명한 패키지로는 [flutter_blue_plus](https://pub.dev/packages/flutter_blue_plus) 가 있습니다. [flutter_blue](https://pub.dev/packages/flutter_blue)가 개발이 중단되어 다른 개발자가 이어서 개발하는 패키지로 알고 있습니다.

이번 포스트에서는 flutter_reactive_ble의 사용방법에 대해서 알아보겠습니다.

---

## Installation

명령어로 설치하는 방법입니다.
`flutter pub add flutter_reactive_ble`

또는 pubspec.yaml 에 입력하여 설치하는 방법입니다.

```
dependencies:
    flutter_reactive_ble: ^5.0.3
```

---

## Features

다음은 pub.dev에 있는 내용입니다.

The reactive BLE lib supports the following:

- BLE device discovery
- Observe host device BLE status
- Establishing a BLE connection
- Maintaining connection status of multiple BLE devices
- Discover services(will be implicit)
- Read / write a characteristic
- Subscribe to a characteristic
- Clear GATT cache
- Negotiate MTU size

---

## Getting Started

Android
안드로이드에서 사용하려면 `AndroidManifest.xml`에 다음의 권한을 추가해야 합니다.

```
<uses-permission android:name="android.permission.BLUETOOTH_SCAN" android:usesPermissionFlags="neverForLocation" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" android:maxSdkVersion="30" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" android:maxSdkVersion="30" />
```

---

## Usage

패키지를 import 합니다.

```
import 'package:flutter_reactive_ble/flutter_reactive_ble.dart';
```

Flutter_reactive_ble 를 사용하기 위해서는 객체를 선언해야 합니다

```
final flutterReactiveBle = FlutterReactiveBle();
```

BLE 디바이스를 검색하여 코드입니다.
results.contains(device) 로 검색을 하면 false만 출력되어 id끼리 비교했습니다.

```
List<DiscoveredDevice> results = [];

flutterReactiveBle.scanForDevices(withServices: []).listen((DiscoveredDevice device) {
    if(results.any((result) => result.id == device.id)) return;
    results.add(device);
});
```

BLE 디바이스와 연결, 해제하는 코드입니다.
Flutter_reactive_ble 패키지에는 디바이스와 연결하는 함수는 있지만 연결을 해제하는 함수는 따로 존재하지 않습니다.
연결할 때 만들어지는 stream을 cancel하는것으로 연결을 해제할 수 있습니다.

```
StreamSubscription<ConnectionStateUpdate> connection;

// 연결
connection = flutterReactiveBle.connectToDevice(id: device.id, connectionTimeout: const Duration(seconds: 4))
            .listen((ConnectionStateUpdate state) {
                switch (state.connectionState) {
                    case DeviceConnectionState.disconnected:
                        break;
                    case DeviceConnectionState.connecting:
                        break;
                    case DeviceConnectionState.connected:
                        break;
                    case DeviceConnectionState.disconnecting:
                        break;
                }
            });

// 해제
connection.cancel();
```

디바이스의 Service와 Chracteristic을 찾아 연결하는 코드입니다.

```
flutterReactiveBle.discoverServices(device.id).then((List<DiscoveredService> services) {
    for(DiscoverdService service in services) {
        for(DiscoveredCharacteristic characteristic in service.characteristics) {
            if(characteristic.isNotifiable) {
                notifyCharacteristic = QualifiedCharacteristic(
                  serviceId: service.serviceId,
                  characteristicId: characteristic.characteristicId,
                  deviceId: device.id,
                );
            }

            if(characteristic.isWritableWithResponse) {
                writeCharacteristic = QualifiedCharacteristic(
                  serviceId: service.serviceId,
                  characteristicId: characteristic.characteristicId,
                  deviceId: device.id,
                );
            }

            if(characteristic.isWritableWithoutResponse) {
            }

            if(characteristic.isReadable) {
                readCharacteristic = QualifiedCharacteristic(
                  serviceId: service.serviceId,
                  characteristicId: characteristic.characteristicId,
                  deviceId: device.id,
                );
            }
        }
    }

    flutterReactiveBle.subscribeToCharacteristic(notifyCharacteristic).listen((value) {
      Logger().i(value);
    });

    flutterReactiveBle.writeCharacteristicWithoutResponse(
        writeCharacteristic,
        value: const Utf8Encoder().convert("Hello World!"),
    );

    flutterReactiveBle.readCharacteristic(readCharacteristic).then((value) {
      Logger().i(value);
    });
});
```
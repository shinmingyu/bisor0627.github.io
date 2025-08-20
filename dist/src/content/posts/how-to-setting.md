---
title: Firebase Analytics vs Amplitude 세팅 가이드
published: 2025-08-20
description: "Flutter 환경에서 Firebase Analytics와 Amplitude를 세팅하는 방법과 실전 예시를 정리한 글입니다."
image: ""
tags: ["Firebase", "Amplitude", "Analytics"]
category: Guide
draft: false
---

# 📊 Firebase Analytics vs Amplitude 개발자를 위한 세팅 가이드

## 🔍 목적

- Firebase Analytics와 Amplitude를 **Flutter 환경에서 어떻게 세팅하는지**
- **설치와 이벤트 트래킹 중심**
- 각 플랫폼의 **기본 코드 예제 중심 정리**

## ⚙️ Firebase Analytics 세팅 (Flutter 기준)

### 1. 패키지 설치

```yaml
dependencies:
  firebase_core: any
  firebase_analytics: ^#.##.#
```

### 2. 초기화 및 인스턴스 준비

```dart
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_analytics/firebase_analytics.dart';

await Firebase.initializeApp(
  options: DefaultFirebaseOptions.currentPlatform,
);

FirebaseAnalytics analytics = FirebaseAnalytics.instance;
```

### 3. 사용자 속성 설정 및 이벤트 로깅

```dart
// 사용자 속성
analytics.setUserProperty(
  name: 'favorite_season',
  value: 'summer',
);

// 이벤트 로깅
analytics.logEvent(
  name: 'preferred_temperature_changed',
  parameters: {
    'preference': 26,
  },
);
```

### 🔧 예시 흐름 (선택 위젯과 연계)

```dart
analytics.setUserProperty(
  name: 'preferred_units',
  value: 'c',
);

analytics.logEvent(
  name: 'hot_or_cold_switch',
  parameters: {'value': 'hot'},
);

analytics.logEvent(
  name: 'rainy_or_sunny_switch',
  parameters: {'rainy_or_sunny_switch': 'sun'},
);
```

📁 참고: `lib/app_state.dart`, `selection_widgets.dart` 내 위젯 연동 구조 확인

## ⚙️ Amplitude 세팅 (Flutter 기준)

### 1. 패키지 설치

```yaml
dependencies:
  amplitude_flutter: # .##.#
```

### 2. 초기화

```dart
import 'package:amplitude_flutter/amplitude.dart';
import 'package:amplitude_flutter/configuration.dart';
import 'package:amplitude_flutter/default_tracking.dart';

final analytics = Amplitude(
  Configuration(
    apiKey: 'YOUR_API_KEY',
    defaultTracking: DefaultTrackingOptions.all(),
  ),
);
```

### 3. 사용자 정보 설정 및 이벤트 로깅

```dart
// 사용자 ID 및 디바이스 ID 설정
analytics.setUserId('user_123');
analytics.setDeviceId('my_device');

// 기본 이벤트 로깅
analytics.track(
  BaseEvent('preferred_temperature_changed'),
);
```

### 4. Identify 사용 (User Property)

```dart
import 'package:amplitude_flutter/events/identify.dart';

final identify = Identify()
  ..set('favorite_season', 'spring')
  ..add('identify_count', 1);

analytics.identify(identify);
```

## 🧪 Flutter 데모 예시 코드 비교

- Firebase:

  - `ApplicationState`를 통해 이벤트 트래킹 관리
  - `SegmentedButton`, `Slider` 등과 연동하여 사용자 입력을 수집

- Amplitude:

  - `DeviceIdForm`, `UserIdForm`, `EventForm` 등 컴포넌트 기반
  - `identify`, `groupIdentify`, `revenue` 등 다양한 이벤트 유형 지원

📁 참고:

- `lib/app_state.dart`, `main.dart`
- `example/lib/my_app.dart`, `event_form.dart`, `identify_form.dart` 등

## ✅ 마무리 요약

| 항목        | Firebase Analytics     | Amplitude                     |
| ----------- | ---------------------- | ----------------------------- |
| 설치 난이도 | 보통                   | 쉬움                          |
| 이벤트 로깅 | `logEvent(...)`        | `track(...)`, `identify(...)` |
| 사용자 속성 | `setUserProperty(...)` | `Identify().set(...)`         |

## 📁 참고 링크 모음

- Firebase 데모 프로젝트: [GA 데모 계정](https://datarian.io/blog/ga-get-demo-account)
- Amplitude 데모 프로젝트: [Flood-it App](https://flood-it.app/)
- Amplitude 공식: [amplitude.com/ko-kr](https://amplitude.com/ko-kr)

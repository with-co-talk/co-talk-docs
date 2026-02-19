---
layout: default
title: Notification API
parent: API Reference
nav_order: 6
---

# 알림/파일 API

푸시 알림, 알림 설정, 파일 업로드를 위한 REST API 문서입니다.

---

## 목차

### 디바이스 토큰
1. [디바이스 토큰 등록](#1-디바이스-토큰-등록)
2. [디바이스 토큰 삭제](#2-디바이스-토큰-삭제)

### 알림 설정
3. [알림 설정 조회](#3-알림-설정-조회)
4. [알림 설정 업데이트](#4-알림-설정-업데이트)

### 파일 업로드
5. [파일 업로드](#5-파일-업로드)

---

## 디바이스 토큰

**Base URL**: `/api/v1/devices`

### 1. 디바이스 토큰 등록

푸시 알림을 위한 FCM/APNs 토큰을 등록합니다.

```http
POST /api/v1/devices/token
Authorization: Bearer {accessToken}
Content-Type: application/json
```

**Request Body**:
```json
{
  "userId": 1,
  "token": "fcm-token-here...",
  "deviceType": "ANDROID"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| userId | Long | O | 사용자 ID |
| token | String | O | FCM/APNs 토큰 |
| deviceType | String | O | 디바이스 타입 (`ANDROID`, `IOS`, `WEB`) |

**Response (201 Created)**:
```json
{
  "tokenId": 100,
  "message": "디바이스 토큰이 등록되었습니다."
}
```

### 디바이스 타입

| 타입 | 설명 |
|------|------|
| ANDROID | Android 디바이스 (FCM) |
| IOS | iOS 디바이스 (APNs/FCM) |
| WEB | 웹 브라우저 (FCM) |

---

### 2. 디바이스 토큰 삭제

로그아웃 시 디바이스 토큰을 삭제합니다.

```http
DELETE /api/v1/devices/token?token={token}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| token | String | O | 삭제할 디바이스 토큰 |

**Response (200 OK)**:
```json
{
  "message": "디바이스 토큰이 삭제되었습니다."
}
```

---

## 알림 설정

**Base URL**: `/api/v1/notifications/settings`

### 3. 알림 설정 조회

사용자의 알림 설정을 조회합니다.

```http
GET /api/v1/notifications/settings?userId={userId}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| userId | Long | O | 사용자 ID |

**Response (200 OK)**:
```json
{
  "userId": 1,
  "messageNotification": true,
  "friendRequestNotification": true,
  "groupInviteNotification": true,
  "soundEnabled": true,
  "vibrationEnabled": true,
  "doNotDisturbEnabled": false,
  "doNotDisturbStart": "23:00",
  "doNotDisturbEnd": "07:00"
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| userId | Long | 사용자 ID |
| messageNotification | Boolean | 메시지 알림 활성화 |
| friendRequestNotification | Boolean | 친구 요청 알림 활성화 |
| groupInviteNotification | Boolean | 그룹 초대 알림 활성화 |
| soundEnabled | Boolean | 소리 활성화 |
| vibrationEnabled | Boolean | 진동 활성화 |
| doNotDisturbEnabled | Boolean | 방해 금지 모드 활성화 |
| doNotDisturbStart | String | 방해 금지 시작 시간 (HH:mm) |
| doNotDisturbEnd | String | 방해 금지 종료 시간 (HH:mm) |

---

### 4. 알림 설정 업데이트

사용자의 알림 설정을 업데이트합니다.

```http
PUT /api/v1/notifications/settings?userId={userId}
Authorization: Bearer {accessToken}
Content-Type: application/json
```

**Request Body**:
```json
{
  "messageNotification": true,
  "friendRequestNotification": true,
  "groupInviteNotification": false,
  "soundEnabled": true,
  "vibrationEnabled": false,
  "doNotDisturbEnabled": true,
  "doNotDisturbStart": "22:00",
  "doNotDisturbEnd": "08:00"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| messageNotification | Boolean | - | 메시지 알림 활성화 |
| friendRequestNotification | Boolean | - | 친구 요청 알림 활성화 |
| groupInviteNotification | Boolean | - | 그룹 초대 알림 활성화 |
| soundEnabled | Boolean | - | 소리 활성화 |
| vibrationEnabled | Boolean | - | 진동 활성화 |
| doNotDisturbEnabled | Boolean | - | 방해 금지 모드 활성화 |
| doNotDisturbStart | String | - | 방해 금지 시작 시간 (HH:mm) |
| doNotDisturbEnd | String | - | 방해 금지 종료 시간 (HH:mm) |

**Response (200 OK)**:
```json
{
  "userId": 1,
  "messageNotification": true,
  "friendRequestNotification": true,
  "groupInviteNotification": false,
  "soundEnabled": true,
  "vibrationEnabled": false,
  "doNotDisturbEnabled": true,
  "doNotDisturbStart": "22:00",
  "doNotDisturbEnd": "08:00"
}
```

---

## 파일 업로드

**Base URL**: `/api/v1/files`

### 5. 파일 업로드

이미지, PDF, 문서 등의 파일을 업로드합니다. 인증된 사용자만 업로드 가능합니다.

```http
POST /api/v1/files/upload
Authorization: Bearer {accessToken}
Content-Type: multipart/form-data
```

**Request Body (Form Data)**:

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| file | File | O | 업로드할 파일 |

**Response (200 OK)**:
```json
{
  "fileUrl": "https://storage.example.com/files/uuid-filename.png",
  "fileName": "original-filename.png",
  "contentType": "image/png",
  "fileSize": 102400,
  "isImage": true
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| fileUrl | String | 업로드된 파일 URL |
| fileName | String | 원본 파일명 |
| contentType | String | MIME 타입 |
| fileSize | Long | 파일 크기 (bytes) |
| isImage | Boolean | 이미지 파일 여부 |

**Error Response**:

**400 Bad Request** - 파일 업로드 실패
```json
{
  "error": "지원하지 않는 파일 형식입니다.",
  "code": "FILE_UPLOAD_ERROR",
  "timestamp": "2025-01-22T10:30:00"
}
```

```json
{
  "error": "파일 크기가 제한을 초과했습니다.",
  "code": "FILE_UPLOAD_ERROR",
  "timestamp": "2025-01-22T10:30:00"
}
```

### 지원 파일 형식

| 카테고리 | MIME 타입 |
|----------|-----------|
| 이미지 | image/jpeg, image/png, image/gif, image/webp |
| 문서 | application/pdf, application/msword, application/vnd.openxmlformats-officedocument.wordprocessingml.document |
| 동영상 | video/mp4, video/quicktime |
| 오디오 | audio/mpeg, audio/wav |

### 파일 크기 제한

| 카테고리 | 최대 크기 |
|----------|-----------|
| 이미지 | 10MB |
| 문서 | 50MB |
| 동영상 | 100MB |

---

## Flutter 연동 예시

### 디바이스 토큰 등록

```dart
import 'package:firebase_messaging/firebase_messaging.dart';

Future<void> registerDeviceToken() async {
  final messaging = FirebaseMessaging.instance;

  // 권한 요청
  await messaging.requestPermission();

  // 토큰 획득
  final token = await messaging.getToken();

  if (token != null) {
    await api.registerDeviceToken(
      userId: currentUserId,
      token: token,
      deviceType: Platform.isIOS ? 'IOS' : 'ANDROID',
    );
  }

  // 토큰 갱신 리스너
  messaging.onTokenRefresh.listen((newToken) {
    api.registerDeviceToken(
      userId: currentUserId,
      token: newToken,
      deviceType: Platform.isIOS ? 'IOS' : 'ANDROID',
    );
  });
}
```

### 파일 업로드

```dart
import 'package:image_picker/image_picker.dart';
import 'package:dio/dio.dart';

Future<FileUploadResponse> uploadImage() async {
  final picker = ImagePicker();
  final image = await picker.pickImage(source: ImageSource.gallery);

  if (image == null) return null;

  final formData = FormData.fromMap({
    'file': await MultipartFile.fromFile(
      image.path,
      filename: image.name,
    ),
  });

  final response = await dio.post(
    '/api/v1/files/upload',
    data: formData,
  );

  return FileUploadResponse.fromJson(response.data);
}
```

### 알림 설정

```dart
// 알림 설정 화면 예시
class NotificationSettingsPage extends StatefulWidget {
  @override
  Widget build(BuildContext context) {
    return FutureBuilder<NotificationSetting>(
      future: api.getNotificationSettings(userId),
      builder: (context, snapshot) {
        if (!snapshot.hasData) return CircularProgressIndicator();

        final settings = snapshot.data!;

        return ListView(
          children: [
            SwitchListTile(
              title: Text('메시지 알림'),
              value: settings.messageNotification,
              onChanged: (value) => updateSetting(
                messageNotification: value,
              ),
            ),
            SwitchListTile(
              title: Text('친구 요청 알림'),
              value: settings.friendRequestNotification,
              onChanged: (value) => updateSetting(
                friendRequestNotification: value,
              ),
            ),
            // ... 기타 설정
          ],
        );
      },
    );
  }
}
```

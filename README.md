# Firebase 구현

project-directory/
│
├── public/                # 웹 애플리케이션 파일 (PWA)
│   ├── index.html         # 메인 HTML 파일
│   ├── styles.css         # CSS 스타일 파일
│   ├── app.js             # 웹 애플리케이션의 주요 JavaScript 파일
│   ├── service-worker.js  # 서비스 워커 파일 (푸시 알림 처리)
│   └── firebase-messaging-sw.js  # Firebase Cloud Messaging 서비스 워커
│
├── backend/               # 백엔드 관련 스크립트
│   ├── [crawler.py](http://crawler.py/)         # 크롤링 스크립트 (Python)
│   └── firestore_utils.py # Firestore와 연동하는 유틸리티 함수
│
├── firebase/              # Firebase 설정 파일
│   ├── firebase.json      # Firebase 배포 설정
│   ├── firestore.rules    # Firestore 보안 규칙
│   ├── .firebaserc        # Firebase 프로젝트 관련 설정
│   └── serviceAccountKey.json # Firebase 서비스 계정 키 (백엔드 전용)
│
├── scripts/               # 배포 및 자동화 스크립트
│   └── [deploy.sh](http://deploy.sh/)          # Firebase 배포 스크립트
│
└── [README.md](http://readme.md/)              # 프로젝트 설명서

## **파일 배치 설명**

### (1) `public/` 폴더 (웹 애플리케이션 관련)

`public/` 폴더는 PWA로 작동하는 웹 애플리케이션의 파일을 저장합니다. Firebase Hosting에서는 기본적으로 이 폴더를 배포 대상으로 설정합니다.

- **`index.html`**: 웹 애플리케이션의 메인 HTML 파일입니다. Firebase Hosting의 기본 진입점입니다.
- **`styles.css`**: PWA의 스타일을 정의합니다.
- **`app.js`**: Firestore 데이터와 UI를 연동하거나, 푸시 알림 기능을 구현하는 메인 스크립트입니다.
- **`service-worker.js`**: 오프라인 지원과 푸시 알림 처리를 위한 PWA 서비스 워커입니다.
- **`firebase-messaging-sw.js`**: Firebase Cloud Messaging 푸시 알림 처리 전용 서비스 워커입니다.

---

### (2) `backend/` 폴더 (백엔드 크롤링 및 Firestore 연동)

`backend/` 폴더는 데이터 크롤링, 가공, Firestore에 저장하는 기능을 담당하는 스크립트를 포함합니다.

- **`crawler.py`**:
    - 네이버 API 또는 Selenium을 이용해 뉴스를 크롤링하는 Python 스크립트입니다.
    - 크롤링 후 데이터를 Firestore에 저장합니다.
    - 정기적으로 실행하기 위해 크론 작업(Cron Job) 또는 Task Scheduler와 연결합니다.
- **`firestore_utils.py`**:
    - Firestore와 데이터를 읽고 쓰는 기능을 모듈화한 스크립트입니다.
    - 예: `save_to_firestore(df, collection_name)` 같은 함수 포함.

---

### (3) `firebase/` 폴더 (Firebase 설정)

`firebase/` 폴더는 Firebase와 관련된 설정 파일을 보관합니다.

- **`firebase.json`**:
    - Firebase Hosting과 Firestore 설정 파일입니다. 예를 들어 `public/` 폴더를 배포 대상으로 지정합니다.
    
    ```json
    json
    코드 복사
    {
      "hosting": {
        "public": "public",
        "ignore": [
          "firebase.json",
          "**/.*",
          "**/node_modules/**"
        ]
      }
    }
    
    ```
    
- **`firestore.rules`**:
    - Firestore의 데이터 보안 규칙을 정의합니다. 예를 들어, 인증된 사용자만 데이터를 읽고 쓰게 설정합니다.
- **`serviceAccountKey.json`**:
    - Python 백엔드에서 Firestore와 통신하기 위한 Firebase 서비스 계정 키입니다.
    - 이 파일은 절대 공개 저장소에 업로드하지 마세요!

---

### (4) `scripts/` 폴더 (자동화 스크립트)

`scripts/` 폴더는 Firebase 배포 또는 정기 작업과 관련된 스크립트를 보관합니다.

- **`deploy.sh`**:
    - Firebase Hosting 및 Firestore 규칙을 자동으로 배포하는 Bash 스크립트입니다.
    
    ```bash
    bash
    코드 복사
    #!/bin/bash
    firebase deploy --only hosting,firestore:rules
    
    ```
    

---

## 3. **개발 및 배포 단계**

1. **크롤링**:
    - `backend/crawler.py`를 정기적으로 실행해 뉴스를 Firestore에 저장합니다.
2. **Firestore 데이터 동기화**:
    - `public/app.js`에서 Firestore와 데이터를 동기화해 UI를 업데이트합니다.
3. **푸시 알림**:
    - 새로운 데이터가 Firestore에 저장되면 `firebase-messaging-sw.js`가 푸시 알림을 처리합니다.
4. **Firebase Hosting 배포**:
    - Firebase CLI를 사용해 PWA를 배포합니다:
        
        ```bash
        bash
        코드 복사
        firebase deploy
        
        ```
        

### 구현

- 기본적인 웹사이트 UI → 해결
- 크롤링 구현 → 네이버 뉴스 API(Python 파일)로 일단 진행
- 웹사이트와 크롤링 연동
    - 크롤링한 데이터를 csv 파일에 저장
    - csv 파일에 있는 것을 Firestore에 저장
    - app.js 파일(동적 데이터를 웹사이트로 불러옴)을 이용해 Firestore의 데이터를 웹사이트와 연동
    - firestore - app.js - index.html 구조

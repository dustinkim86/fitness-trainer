# iOS 앱스토어 등록 가이드

## 준비물 (사전 완료 필요)

### 1. Apple Developer 계정 등록
- https://developer.apple.com/programs/ 에서 연 $99(약 13만원) 결제
- 이메일 인증 후 24시간 내 승인

### 2. App Store Connect 앱 등록
1. https://appstoreconnect.apple.com 접속
2. "나의 앱" → "+" → "새로운 앱"
3. 아래 정보 입력:
   - 이름: JH Fitness
   - 기본 언어: 한국어
   - 번들 ID: com.junho.fitness
   - SKU: jh-fitness-001

---

## 방법 A: Mac에서 직접 빌드 (권장)

### 단계 1: 환경 준비
```bash
# Mac에서 실행
# Xcode 설치 (App Store에서 무료)
# CocoaPods 설치
sudo gem install cocoapods
# Node.js 설치 (https://nodejs.org)
```

### 단계 2: 프로젝트 클론
```bash
git clone https://github.com/dustinkim86/fitness-trainer.git
cd fitness-trainer
npm install
```

### 단계 3: CocoaPods 설치 및 Xcode 열기
```bash
cd ios/App
pod install
cd ../..
npx cap open ios   # Xcode가 자동으로 열림
```

### 단계 4: Xcode 서명 설정
1. Xcode에서 `App` 타겟 선택
2. "Signing & Capabilities" 탭
3. "Team" → Apple Developer 계정 선택
4. Bundle Identifier: `com.junho.fitness` 확인

### 단계 5: App Store 아카이브 생성
1. Xcode 상단: 디바이스를 "Any iOS Device (arm64)"로 설정
2. 메뉴: Product → Archive
3. 완료 후 Organizer 창이 열림
4. "Distribute App" → "App Store Connect" 선택
5. 안내에 따라 업로드

### 단계 6: App Store Connect에서 제출
1. https://appstoreconnect.apple.com 에서 빌드 확인 (약 30분 소요)
2. 앱 정보 입력:
   - 스크린샷 (iPhone 6.5인치 필수)
   - 앱 설명 (한국어)
   - 키워드: 운동,헬스,피트니스,트레이너,영양제
3. "검토를 위해 제출" 클릭
4. 심사 기간: 보통 1~3일

---

## 방법 B: GitHub Actions 자동 빌드

### 필요한 GitHub Secrets 설정
GitHub 저장소 → Settings → Secrets → Actions 에서 아래 항목 추가:

| Secret 이름 | 값 |
|------------|---|
| `BUILD_CERTIFICATE_BASE64` | .p12 인증서를 base64 인코딩한 값 |
| `P12_PASSWORD` | .p12 파일 비밀번호 |
| `BUILD_PROVISION_PROFILE_BASE64` | .mobileprovision 파일을 base64 인코딩한 값 |
| `KEYCHAIN_PASSWORD` | 임의의 안전한 비밀번호 |
| `PROVISIONING_PROFILE_NAME` | 프로비저닝 프로파일 이름 |
| `APP_STORE_CONNECT_USERNAME` | Apple ID 이메일 |
| `APP_STORE_CONNECT_PASSWORD` | 앱 전용 비밀번호 (https://appleid.apple.com) |

### 인증서 base64 인코딩 방법 (Mac Terminal)
```bash
base64 -i Certificates.p12 | pbcopy   # 클립보드에 복사됨
base64 -i profile.mobileprovision | pbcopy
```

### 빌드 트리거
```bash
# 태그 생성 시 자동으로 빌드+업로드
git tag v1.0.0
git push origin v1.0.0
```

---

## 앱스토어 심사 팁

### 거절 방지 주의사항
- 앱 설명에 "웹사이트"라는 표현 사용 금지
- 개인 데이터 수집 시 개인정보처리방침 URL 필수
- 로그인 없이도 일부 기능 동작해야 함 (현재 앱은 통과 가능)

### 권장 앱 설명 (한국어)
```
JH Fitness는 개인 헬스 트레이너 앱입니다.

주요 기능:
• 운동 루틴 관리 및 세트/무게 기록
• 식단 체크 (운동일/휴식일 구분)
• 영양제 복용 일정 관리
• 모빌리티 스트레칭 가이드
• 운동 기록 달력 및 히스토리
• 휴식 타이머

개인 피트니스 목표 달성을 위한 올인원 트레이닝 도우미입니다.
```

---

## 프로젝트 구조
```
fitness-trainer/
├── index.html          # 메인 앱 소스
├── www/                # iOS 번들링용 (Capacitor 사용)
│   ├── index.html      # 로컬 스크립트로 변환된 버전
│   ├── react.min.js
│   ├── react-dom.min.js
│   └── babel.min.js
├── ios/                # Xcode 프로젝트 (Capacitor 생성)
├── capacitor.config.json
└── .github/workflows/ios-build.yml
```

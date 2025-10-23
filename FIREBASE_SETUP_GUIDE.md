# Firebase 실시간 점수 집계 시스템 설정 가이드

## 🚀 Firebase 프로젝트 설정

### 1단계: Firebase 프로젝트 생성

1. [Firebase Console](https://console.firebase.google.com/) 접속
2. "프로젝트 추가" 클릭
3. 프로젝트 이름 입력 (예: `realtime-scoring`)
4. Google Analytics 설정 (선택사항)
5. 프로젝트 생성 완료

### 2단계: 웹 앱 등록

1. Firebase 프로젝트 개요 페이지에서 웹 아이콘(`</>`) 클릭
2. 앱 닉네임 입력 (예: `점수집계앱`)
3. "Firebase 호스팅도 설정합니다" 체크 해제 (선택사항)
4. "앱 등록" 클릭

### 3단계: Firebase SDK 구성 정보 복사

앱 등록 후 표시되는 구성 객체를 복사합니다:

```javascript
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "yourapp.firebaseapp.com",
  databaseURL: "https://yourapp-default-rtdb.firebaseio.com",
  projectId: "yourapp",
  storageBucket: "yourapp.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc..."
};
```

**필요한 정보:**
- `apiKey`
- `authDomain`
- `databaseURL`
- `projectId`

### 4단계: Realtime Database 생성

1. Firebase Console 왼쪽 메뉴에서 "Realtime Database" 선택
2. "데이터베이스 만들기" 클릭
3. 데이터베이스 위치 선택 (아시아: `asia-southeast1` 권장)
4. 보안 규칙 선택:
   - **테스트 모드로 시작** (개발/테스트용)
   - **또는 잠금 모드로 시작** (아래 규칙 설정 필요)

### 5단계: 보안 규칙 설정

#### 개발/테스트용 (누구나 읽기/쓰기 가능)

```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

⚠️ **주의**: 프로덕션 환경에서는 절대 사용하지 마세요!

#### 프로덕션 권장 (읽기 전용, 쓰기는 인증 필요)

```json
{
  "rules": {
    ".read": true,
    "songs": {
      ".write": "auth != null"
    },
    "voters": {
      ".write": true
    },
    "votingStatus": {
      ".write": "auth != null"
    },
    "results": {
      "$songId": {
        "votes": {
          ".write": true
        }
      }
    }
  }
}
```

#### 더 강력한 보안 규칙 (중복 투표 방지)

```json
{
  "rules": {
    ".read": true,
    "songs": {
      "$songId": {
        ".write": "auth != null",
        ".validate": "newData.hasChildren(['title', 'artist', 'performer'])"
      }
    },
    "votingStatus": {
      ".write": "auth != null"
    },
    "results": {
      "$songId": {
        "votes": {
          "$voteId": {
            ".write": "!data.exists()",
            ".validate": "newData.hasChildren(['voter', 'score', 'timestamp']) && newData.child('score').val() >= 1 && newData.child('score').val() <= 10"
          }
        }
      }
    }
  }
}
```

### 6단계: 앱에 설정 정보 입력

1. `realtime-scoring.html` 파일을 브라우저에서 열기
2. Firebase 설정 섹션에 정보 입력:
   - API Key
   - Auth Domain
   - Database URL
   - Project ID
3. "연결하기" 클릭

## 📱 사용 방법

### 관리자 모드

1. "관리자 모드" 선택
2. 곡 정보 입력 및 추가
3. "현재 곡으로" 버튼으로 평가할 곡 선택
4. 실시간 결과 확인

### 참여자 모드

1. "참여자 모드" 선택
2. 이름 입력
3. 현재 평가 곡 확인
4. 1~10점 선택 후 "투표하기"

### 결과 화면 모드

1. "결과 화면" 선택
2. 실시간으로 업데이트되는 순위 확인
3. 대형 화면에 표시하여 모든 참여자가 볼 수 있도록 활용

## 🎯 주요 기능

### ✅ 실시간 동기화
- 모든 기기에서 즉시 반영
- 점수 입력 즉시 순위 업데이트

### ✅ 다중 접속 지원
- 무제한 참여자 동시 접속
- 관리자/참여자/결과화면 분리

### ✅ 연결 상태 표시
- 우측 상단에 실시간 연결 상태 표시
- 인터넷 연결 끊김 시 자동 감지

### ✅ 자동 저장
- Firebase 설정 정보 로컬 저장
- 투표자 이름 자동 기억

## 🔧 문제 해결

### "Firebase 연결 실패" 오류

1. **Database URL 확인**
   - `https://yourapp-default-rtdb.firebaseio.com` 형식인지 확인
   - Firebase Console > Realtime Database에서 URL 복사

2. **API Key 확인**
   - 공백이나 따옴표 없이 정확히 복사

3. **보안 규칙 확인**
   - Firebase Console > Realtime Database > 규칙 탭
   - 테스트 모드로 설정되어 있는지 확인

### 데이터가 저장되지 않음

1. **보안 규칙 점검**
   ```json
   {
     "rules": {
       ".read": true,
       ".write": true
     }
   }
   ```

2. **브라우저 콘솔 확인**
   - F12 눌러 개발자 도구 열기
   - Console 탭에서 에러 메시지 확인

3. **인터넷 연결 확인**
   - 우측 상단 연결 상태 확인

### 실시간 업데이트가 안됨

1. **페이지 새로고침**
2. **다른 브라우저에서 테스트**
3. **Firebase Console에서 데이터 직접 확인**

## 💡 팁

### 여러 기기에서 사용하기

1. **같은 WiFi 네트워크 연결**
2. **HTML 파일 공유 방법:**
   - 파일 공유 서버 사용
   - Google Drive/Dropbox 공유
   - GitHub Pages 호스팅

### GitHub Pages로 호스팅하기

```bash
# Git 저장소 초기화
git init
git add realtime-scoring.html
git commit -m "Add realtime scoring system"

# GitHub 저장소 생성 후
git remote add origin https://github.com/username/repo.git
git branch -M main
git push -u origin main

# Settings > Pages에서 배포 설정
# https://username.github.io/repo/realtime-scoring.html 로 접속
```

### 로컬 서버로 테스트하기

```bash
# Python 3
python -m http.server 8000

# Node.js (http-server 설치 필요)
npx http-server

# 브라우저에서 http://localhost:8000/realtime-scoring.html 접속
```

## 🔐 보안 강화

### Firebase Authentication 추가 (선택사항)

관리자 기능을 보호하려면 Firebase Authentication 설정:

1. Firebase Console > Authentication > 시작하기
2. 이메일/비밀번호 활성화
3. 보안 규칙 수정하여 인증된 사용자만 관리 기능 사용

### 환경 변수로 설정 보호

민감한 정보는 환경 변수로 관리:

```javascript
// .env 파일 생성
VITE_FIREBASE_API_KEY=your_api_key
VITE_FIREBASE_AUTH_DOMAIN=your_auth_domain
// ...

// 빌드 도구 사용 시 환경 변수 주입
```

## 📊 데이터 구조

Firebase Realtime Database 구조:

```
{
  "songs": {
    "songId1": {
      "title": "아무노래",
      "artist": "지코",
      "performer": "홍길동",
      "timestamp": 1234567890
    }
  },
  "votingStatus": {
    "currentSongId": "songId1",
    "isVoting": true,
    "timestamp": 1234567890
  },
  "results": {
    "songId1": {
      "votes": {
        "voteId1": {
          "voter": "김철수",
          "score": 9,
          "timestamp": 1234567890
        }
      }
    }
  }
}
```

## 🆘 지원

문제가 발생하면:

1. [Firebase 공식 문서](https://firebase.google.com/docs/database)
2. [Stack Overflow - firebase](https://stackoverflow.com/questions/tagged/firebase)
3. 브라우저 콘솔에서 에러 메시지 확인

---

**즐거운 행사 되세요!** 🎉

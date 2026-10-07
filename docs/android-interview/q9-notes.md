# Q9. Service란 무엇인가요? - 내 답안 노트

- 원문: `manifest-android-interview-kr.pdf` 52~62쪽 (PDF 60~70쪽)

## 실전 질문

> Q) 안드로이드에서 Started 서비스와 Bound 서비스의 차이점은 무엇이며, 각각 언제 사용해야 하나요?

## 내 답안

TODO(human): 면접에서 말하듯 5~10줄로 작성

## 학습 메모 (사이클 요약)

근거: 원문 52~62쪽, 공식 문서 [Services overview](https://developer.android.com/develop/background-work/services), [Bound services overview](https://developer.android.com/develop/background-work/services/bound-services), [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager), [Persistent work](https://developer.android.com/develop/background-work/background-tasks/persistent). "원문 밖"은 책에 없는 내용.

### 1. Service의 정체
- 사용자 인터페이스(user interface)가 없는 앱 컴포넌트. 앱이 백그라운드 상태(background state)여도 실행될 수 있다.
- (원문 밖) 메인 스레드(main thread)에서 실행된다. 작업자 스레드(worker thread)는 직접 만들어야 한다. 메인 스레드가 입력에 5초 넘게 응답하지 못하면 ANR.

### 2. Started
- `startService()`로 시작, `stopSelf()` 또는 `stopService()`로 종료. 시작한 컴포넌트와 독립적.
- 자동으로 멈추지 않으므로 작업이 끝나면 직접 멈춰야 한다(리소스 낭비 방지).
- (원문 밖) Started/Bound는 Service의 종류가 아니라 시작 방식. 한 Service가 두 상태를 동시에 가질 수 있고, 그때는 stop 호출 + 모든 클라이언트 unbind가 모두 있어야 종료.

### 3. Bound
- `bindService()`로 연결, `onBind()`가 `IBinder`(통로) 반환. Service = 서버, 바인딩한 컴포넌트 = 클라이언트.
- 모든 클라이언트가 unbind하면 자동 종료(started 상태가 아닐 때).
- 고르는 기준: 통신 방향(보내고 끝 vs 양방향), 수명을 정하는 주체(독립 vs 클라이언트에 종속).

### 4. Foreground
- 지속적인 알림을 표시하는 Service. 사용자 인지, 높은 우선순위, 사용자가 계속 알아야 하는 작업.
- `startForeground()` 호출 시 즉시 알림. Android 8.0+ 알림 채널, Android 14+ 포그라운드 서비스 유형(매니페스트 + 런타임) 필수, 없으면 `MissingForegroundServiceTypeException`.
- Started/Bound와 기준이 다르다(우선순위와 알림 표시). 보통 started 상태에서 승격.
- (원문 밖) Android 8.0+ 백그라운드 상태에서 `startService()` → `IllegalStateException`. `startForegroundService()` 후 5초 안에 `startForeground()`.

### 5. 생명주기
- started: `onCreate` → `onStartCommand` → `onDestroy`. bound: `onCreate` → `onBind` → `onUnbind` → `onDestroy`.
- (원문 밖) `startService()`, `bindService()`는 즉시 반환(비동기). `onCreate`는 이미 실행 중이면 호출 안 됨. `onStartCommand`는 매번, `onBind`는 첫 클라이언트 때 한 번(IBinder 캐시).
- `onUnbind()`가 `true`면 다음 바인딩 때 `onRebind()`. started이면서 bound인 Service가 살아 있을 때 클라이언트가 돌아온 신호.
- `START_NOT_STICKY` 재생성 안 함 / `START_STICKY` 재생성, intent `null` / `START_REDELIVER_INTENT` 마지막 Intent 재전달.
- Activity 생명주기는 화면 가시성이, Service 생명주기는 다른 컴포넌트의 호출이 움직인다.

### 6. WorkManager (원문 밖, 원문은 권고만)
- 작업을 내부 SQLite에 저장 → 앱 재시작, 기기 재부팅 뒤에도 다시 예약. 실행 조건, 지수 백오프 재시도, 코루틴 연동 제공.
- 어울리는 작업: 로그/분석 데이터 전송, 서버와 주기적 데이터 동기화, 이미지 처리 후 업로드.
- 안 맞는 작업: 프로세스가 사라지면 함께 끝나도 되는 작업(코루틴으로 충분), 즉시 실행이 필요한 모든 작업.
- 고르는 순서: 화면 닫히면 끝나도 됨 → 코루틴 / 사용자가 지금 보고 듣는 작업 → Foreground Service / 늦어도 되지만 반드시 끝나야 함 → WorkManager / 화면이 Service와 계속 대화 → Bound Service.

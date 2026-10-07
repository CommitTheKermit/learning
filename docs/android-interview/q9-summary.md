# Q9. Service 요약

## 1. Service란

- 사용자 인터페이스(user interface)가 없는 앱 컴포넌트.
- 앱이 백그라운드 상태(background state)여도 실행될 수 있음.
- **함정**: 실행은 메인 스레드(main thread)에서 함. 작업자 스레드(worker thread)는 직접 만들어야 함.
- 메인 스레드를 5초 넘게 막으면 ANR 발생.

> "백그라운드 상태에서 살아 있을 수 있지만, 실행은 메인 스레드에서 한다."

---

## 2. Started vs Bound

| 기준 | Started | Bound |
| --- | --- | --- |
| 시작 | `startService()` | `bindService()` |
| 통신 방향 | 한 방향. Intent를 보내고 끝 | 양방향. `IBinder`로 메서드 직접 호출 |
| 수명을 정하는 주체 | Service 자신이 `stopSelf()` 혹은 다른 컴포넌트가 `stopService()`로 종료 | 클라이언트. 마지막 클라이언트가 unbind하면 종료 |
| 사용 예 | 화면을 닫아도 계속되는 작업 | 화면이 Service에 계속 묻는 작업 |

- Started/Bound는 Service의 종류가 아니라 **시작 방식**.
- 한 Service가 두 상태를 동시에 가질 수 있음. 이때는 stop 호출과 모든 unbind가 모두 있어야 종료.
- 예: 음악 앱은 **시작**해서 재생하고, 화면이 돌아오면 **바인딩**해서 제어함.

---

## 3. Foreground Service

- 지속적인 알림(ongoing notification)을 표시하며 실행되는 Service.
- `startForeground()` 호출 시 즉시 알림 표시. 시스템이 종료할 가능성이 적음.
- Started/Bound와 기준이 다름(우선순위와 알림 표시). 보통 started 상태에서 승격.
- Android 8.0+: 백그라운드 상태에서 `startService()` → `IllegalStateException`. `startForegroundService()` 후 5초 안에 `startForeground()`.
- Android 14+: 포그라운드 서비스 유형(foreground service type) 필수. 없으면 `MissingForegroundServiceTypeException`.

![원문 55쪽 Service 유형 간 차이점 표](https://raw.githubusercontent.com/CommitTheKermit/learning/main/docs/android-interview/images/q9/book-p55-service-types-table.png)

*원문 55쪽 표. 세 가지를 같은 "유형"처럼 나란히 놓았지만, Started/Bound는 시작 방식이고 Foreground는 우선순위와 알림 표시라는 다른 기준임.*

---

## 4. 생명주기

![Service 생명주기](https://raw.githubusercontent.com/CommitTheKermit/learning/main/docs/android-interview/images/q9/book-fig31-service-lifecycle.png)

*출처: 원문 59쪽 그림 31*

- `onCreate()`: 처음 생성될 때 한 번.
- `onStartCommand()`: `startService()` 호출마다 **매번**.
- `onBind()`: **첫 클라이언트** 때 한 번. 이후 클라이언트는 같은 `IBinder`를 받음.
- `onUnbind()`가 `true` 반환 → 살아 있는 Service에 클라이언트가 돌아오면 `onRebind()`.

![started이면서 bound인 Service의 생명주기](https://developer.android.com/static/images/fundamentals/service_binding_tree_lifecycle.png)

*출처: Android Developers, Bound services overview. started이면서 bound인 Service의 흐름 (`onRebind()` 포함)*

| `onStartCommand()` 반환값 | 강제 종료 후 |
| --- | --- |
| `START_NOT_STICKY` | 다시 생성 안 함 |
| `START_STICKY` | 다시 생성, intent는 `null` |
| `START_REDELIVER_INTENT` | 다시 생성, 마지막 Intent 재전달 |

---

## 5. WorkManager

- 원문: 즉시 실행할 필요가 없는 작업은 Service 대신 WorkManager.
- 작업을 SQLite에 저장 → 앱 재시작, 기기 재부팅 뒤에도 다시 예약됨.
- 실행 조건(constraint), 재시도(retry), 스레드 처리를 라이브러리가 제공.
- Service는 프로세스가 죽거나 재부팅되면 작업이 사라짐.

---

## 6. 무엇을 쓸지 고르는 순서

```
화면이 닫히면 끝나도 되는가? ── 예 ──▶ 코루틴
        │ 아니요
사용자가 지금 보고 듣는 작업인가? ── 예 ──▶ Foreground Service
        │ 아니요
늦어도 되지만 반드시 끝나야 하는가? ── 예 ──▶ WorkManager

※ 화면이 Service와 계속 대화해야 하면 ──▶ Bound Service
```

### 상태 조합별 예

Foreground는 혼자 설 수 없음. `startForegroundService()`로 **시작**한 뒤 `startForeground()`로 **승격**하므로 started 상태 위에 올라감.

| 조합 | 어울리는 예 | 이유 |
| --- | --- | --- |
| Started + Bound + Foreground | 내비게이션, 음악 재생 | 화면을 꺼도 계속(Started), 사용자가 진행을 알아야 함(Foreground), 화면이 위치·재생 상태를 계속 물어봄(Bound) |
| Started + Foreground | 사용자가 누른 대용량 파일 업로드 (진행률은 알림으로만) | 앱을 닫아도 끝나야 하고 사용자가 알아야 함. 화면과 대화할 일은 없음 |
| Bound만 | 블루투스 기기 설정 화면, 다른 앱·시스템에 기능을 제공하는 Service | 화면이 열린 동안만 의미가 있음. 바인딩은 백그라운드 실행 제한의 영향을 받지 않음 |
| Started + Bound | 앱이 열린 동안의 큰 작업 + 진행률 화면 | 범위가 좁음. Foreground 없는 started Service는 앱이 백그라운드로 가고 몇 분 뒤 시스템이 멈춤 |
| Started만 | 앱이 화면에 있는 동안 끝나는 짧은 작업 | 범위가 좁음. 급하지 않은 작업은 WorkManager가 대신함 |
| Bound + Foreground | - | 공식 시작 흐름 밖(문서 설명 없음). bound만인 Service는 마지막 화면이 끊으면 종료되므로 "화면 없이 살아남기"라는 Foreground의 목적과 어긋남 (추론) |

- 고르는 세 질문: 앱을 닫아도 계속되는가(Started), 사용자가 알아야 하는가(Foreground), 화면이 계속 묻는가(Bound).

---

### 실전 질문: 안드로이드에서 Started 서비스와 Bound 서비스의 차이점은 무엇이며, 각각 언제 사용해야 하나요?

## 7. 용어집

| 용어 | 뜻 |
| --- | --- |
| 앱 컴포넌트(app component) | 시스템이 앱에 들어오는 진입점. Activity, Service 등 |
| 백그라운드 상태(background state) | 사용자가 앱 화면을 보지 않는 상태 |
| 메인 스레드(main thread) | 화면을 그리고 터치를 처리하는 스레드. UI 스레드라고도 부름 |
| 작업자 스레드(worker thread) | 메인 스레드가 아닌 스레드. Service에서는 직접 만들어야 함 |
| ANR(Application Not Responding) | 메인 스레드가 입력에 5초 넘게 응답하지 못할 때 뜨는 오류 |
| 리소스 낭비(resource waste) | 끝난 Service가 살아 있어 메모리, 배터리, CPU를 쓰는 것 |
| started 상태 | `startService()`로 시작된 상태. 시작한 컴포넌트와 독립적 |
| bound 상태 | `bindService()`로 연결된 상태. 클라이언트가 있는 동안만 유지 |
| 클라이언트(client) | Service에 바인딩한 컴포넌트. Service는 서버(server) |
| `IBinder` | 클라이언트와 Service 사이의 통로(interface). `onBind()`가 반환 |
| `ServiceConnection` | 바인딩 결과(`IBinder`)를 받는 클라이언트 쪽 콜백 |
| 비동기(asynchronous) | 호출이 즉시 반환되고 결과는 나중에 오는 방식 |
| `onRebind()` | started이면서 bound인 Service에 클라이언트가 돌아올 때 호출 |
| Foreground Service | 지속적인 알림을 표시하며 실행되는 Service |
| 지속적인 알림(ongoing notification) | 작업이 진행 중임을 알리며 알림 영역에 계속 표시되는 알림 |
| 알림 채널(notification channel) | Android 8.0+ 에서 알림을 띄우기 전에 만들어야 하는 분류 |
| 포그라운드 서비스 유형(foreground service type) | Android 14+ 에서 필수인 작업 종류 선언. 예: `dataSync`, `mediaPlayback` |
| `START_STICKY` 등 | `onStartCommand()` 반환값. 강제 종료 뒤 다시 생성할지 정함 |
| WorkManager | 반드시 끝나야 하지만 급하지 않은 작업용 Jetpack 라이브러리 |
| 지속성 작업(persistent work) | 앱 재시작, 기기 재부팅 뒤에도 다시 예약되는 작업 |
| 실행 조건(constraint) | 작업이 실행될 조건. 예: 네트워크 연결됨 |
| 지수 백오프(exponential backoff) | 실패할 때마다 재시도 간격을 늘리는 정책 |
| 코루틴(coroutine) | Kotlin의 비동기 처리 도구. 화면과 함께 끝나도 되는 작업에 사용 |

---

출처: 『매니페스트 안드로이드 인터뷰』 52~62쪽, Android Developers [Services overview](https://developer.android.com/develop/background-work/services), [Bound services overview](https://developer.android.com/develop/background-work/services/bound-services), [WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager), [Launch a foreground service](https://developer.android.com/develop/background-work/services/fgs/launch), [Background execution limits](https://developer.android.com/about/versions/oreo/background)

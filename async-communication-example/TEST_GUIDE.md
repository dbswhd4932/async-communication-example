# 성능 테스트 가이드

## 테스트 개요

동기 방식과 비동기 방식의 성능 차이를 측정하는 테스트입니다.

## 테스트 파일 위치

- **테스트 서비스**: `src/test/java/com/example/async/service/PerformanceTestService.java`
- **성능 테스트**: `src/test/java/com/example/async/service/SyncVsAsyncPerformanceTest.java`

## 테스트 실행 방법

### 1. IDE에서 실행 (권장)

IntelliJ IDEA 또는 Eclipse에서:
1. `SyncVsAsyncPerformanceTest.java` 파일 열기
2. 클래스 왼쪽의 실행 버튼 클릭 또는
3. 개별 테스트 메서드 실행

### 2. Gradle로 실행

```bash
# 모든 테스트 실행
./gradlew test

# 특정 테스트만 실행
./gradlew test --tests SyncVsAsyncPerformanceTest

# 상세 로그와 함께 실행
./gradlew test --tests SyncVsAsyncPerformanceTest --info
```

## 테스트 시나리오

### 1️⃣ 동기 방식 성능 테스트

**메서드**: `testSynchronousPerformance()`

**시나리오**:
- 5개 작업을 순차적으로 실행
- 각 작업은 1초씩 소요
- **예상 소요 시간**: 약 5초

**실행 로그 예시**:
```
=== 동기 방식 성능 테스트 시작 ===
[main] Starting sync task: Task-1
[main] Completed sync task: Task-1
[main] Starting sync task: Task-2
[main] Completed sync task: Task-2
[main] Starting sync task: Task-3
[main] Completed sync task: Task-3
[main] Starting sync task: Task-4
[main] Completed sync task: Task-4
[main] Starting sync task: Task-5
[main] Completed sync task: Task-5
=== 동기 방식 성능 테스트 완료 ===
총 소요 시간: 5025ms (5.025초)
✅ 동기 방식 검증 완료: 순차 실행으로 5025ms 소요
```

---

### 2️⃣ 비동기 방식 성능 테스트

**메서드**: `testAsynchronousPerformance()`

**시나리오**:
- 5개 작업을 **병렬로** 실행
- 각 작업은 1초씩 소요하지만 동시에 실행
- **예상 소요 시간**: 약 1초 (병렬 처리)

**실행 로그 예시**:
```
=== 비동기 방식 성능 테스트 시작 ===
[async-1] Starting async task: Task-1
[async-2] Starting async task: Task-2
[async-3] Starting async task: Task-3
[async-4] Starting async task: Task-4
[async-5] Starting async task: Task-5
[async-1] Completed async task: Task-1
[async-2] Completed async task: Task-2
[async-3] Completed async task: Task-3
[async-4] Completed async task: Task-4
[async-5] Completed async task: Task-5
=== 비동기 방식 성능 테스트 완료 ===
총 소요 시간: 1035ms (1.035초)
✅ 비동기 방식 검증 완료: 병렬 실행으로 1035ms 소요
```

---

### 3️⃣ 동기 vs 비동기 성능 비교

**메서드**: `testSyncVsAsyncComparison()`

**시나리오**:
- 10개 작업으로 동기/비동기 방식 비교
- 동기: 약 10초 소요
- 비동기: 약 1-2초 소요
- 성능 개선율 계산

**실행 로그 예시**:
```
============================================================
동기 vs 비동기 성능 비교 테스트 시작
============================================================

[1] 동기 방식 - 10개 작업 순차 실행
동기 방식 소요 시간: 10050ms (10.05초)

[2] 비동기 방식 - 10개 작업 병렬 실행
비동기 방식 소요 시간: 1028ms (1.028초)

============================================================
📊 성능 비교 결과
============================================================
작업 개수: 10개 (각 작업당 1초 소요)
동기 방식 소요 시간:   10050ms (10.05초)
비동기 방식 소요 시간: 1028ms (1.028초)
단축된 시간: 9022ms (9.022초)
성능 개선율: 89.77%
============================================================
✅ 성능 비교 검증 완료
   - 동기는 10개 작업을 순차 실행하여 약 10초 소요
   - 비동기는 10개 작업을 병렬 실행하여 약 1-2초 소요
   - 비동기 방식이 약 89.77% 더 빠름
```

## 성능 비교 요약

| 구분 | 작업 수 | 동기 방식 | 비동기 방식 | 성능 개선 |
|------|---------|-----------|-------------|-----------|
| 테스트 1 | 5개 | ~5초 | ~1초 | 80% |
| 테스트 2 | 10개 | ~10초 | ~1초 | 90% |

## 핵심 포인트

### 동기 방식
```java
// 순차 실행 - 각 작업이 끝날 때까지 대기
for (int i = 1; i <= 5; i++) {
    performanceTestService.syncTask("Task-" + i);
}
// 총 소요 시간 = 1초 × 5개 = 5초
```

### 비동기 방식
```java
// 병렬 실행 - 모든 작업을 동시에 시작
List<CompletableFuture<String>> futures = new ArrayList<>();
for (int i = 1; i <= 5; i++) {
    futures.add(performanceTestService.asyncTask("Task-" + i));
}
CompletableFuture.allOf(futures.toArray(new CompletableFuture[0])).join();
// 총 소요 시간 = 1초 (병렬 처리)
```

## 스레드 확인

로그에서 스레드 이름을 확인하면:

- **동기 방식**: `[main]` - 메인 스레드에서 순차 실행
- **비동기 방식**: `[async-1]`, `[async-2]`, `[async-3]` - 여러 스레드에서 병렬 실행

## 테스트 검증 조건

### 동기 방식 검증
```java
// 5개 작업 * 1초 = 최소 5초 이상
assertTrue(duration >= 5000, "동기 방식은 최소 5초 이상 소요되어야 함");
```

### 비동기 방식 검증
```java
// 병렬 처리로 1-2초 사이
assertTrue(duration >= 1000, "비동기 방식은 최소 1초 이상 소요되어야 함");
assertTrue(duration < 2000, "비동기 방식은 2초 미만이어야 함");
```

## 문제 해결

### Gradle 빌드 실패 시
```bash
# Gradle wrapper 재생성
gradle wrapper --gradle-version 8.5

# 또는 IDE에서 직접 실행 (권장)
```

### 테스트가 너무 느릴 때
- 작업 개수를 줄여서 테스트 (5개 → 3개)
- Thread.sleep 시간 조정 (1000ms → 500ms)

## 실무 적용

이 테스트 결과는 다음과 같은 실무 상황에 적용됩니다:

1. **이메일 대량 발송**: 100명에게 이메일 → 동기: 100초, 비동기: 1-2초
2. **외부 API 호출**: 10개 API 동시 호출 → 동기: 10초, 비동기: 1초
3. **파일 처리**: 여러 파일 동시 처리 → 성능 대폭 개선
4. **데이터베이스 조회**: 여러 테이블 병렬 조회 → 응답 시간 단축

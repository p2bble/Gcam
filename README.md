# Unihertz Titan 2 Elite GCam – 초점 링 위치 패치

[Illuminati Elite GCam v1.4](https://www.reddit.com/r/unihertz/comments/1vzxax6/gcam_port_for_the_unihertz_titan_2_elite_square/) (Hasli LMC8.4 R18 기반)에서
**탭한 곳보다 초점 링이 62px 아래에 그려지던 문제**를 고친 패치입니다.
실제 초점은 원래부터 정확했고, 표시만 어긋나 있었습니다.

- 기기: Unihertz Titan 2 Elite (Dimensity 7400, 1076x1200, Android 16 V02.00.04)
- 원본: `IlluminatiEliteGCam_v1.4_Titan2Elite.apk` (패키지 `com.google.android.GoogleCameraEngR18F1`)
- 바뀐 파일: smali 1개, 실질 1줄 (`patch/focus-ring-offset.patch`)

## 설치

1. 기존 GCam 포트를 제거합니다. 서명이 달라 덮어쓰기 설치가 안 됩니다.
2. [Releases](../../releases)에서 `IlluminatiEliteGCam_v1.4_Titan2Elite_focusfix.apk`를 받아 설치합니다.
3. Play Protect가 막으면 "자세히 보기 → 무시하고 설치"를 누릅니다. 새 서명이라 처음 한 번 뜹니다.
4. 카메라·마이크 권한을 줍니다.

## 증상과 원인

| | 패치 전 | 패치 후 |
|---|---|---|
| 뷰파인더(SurfaceView) | y 0~1200 | y 0~1200 |
| 오버레이(초점 링·격자) | y 62~1200 | y 0~1200 |
| (300,300) 탭 → 링 중심 | (300,362) | (300,300) |
| (900,700) 탭 → 링 중심 | (900,762) | (900,700) |

`CameraAppRootLayout.onLayout`이 화면이 짧으면(SIMPLIFIED_LAYOUT) 상태바 높이(이 기기 123px)를 위쪽 패딩으로 넣습니다.
구글 원본은 상태바를 보여 주므로 맞는 동작이지만, 이 포트는 상태바를 숨깁니다.
남은 패딩 때문에 1200px 오버레이가 1077px 영역에 가운데 정렬되면서 (1200-1077)/2 ≈ 62px 밀렸습니다.
패치는 그 패딩 분기를 건너뜁니다. 상단 바(설정·플래시)는 패딩과 무관하게 자리를 잡아 위치가 그대로입니다.

## 직접 빌드

필요: JDK 17, [apktool 2.10](https://github.com/iBotPeaches/Apktool/releases), [uber-apk-signer 1.3](https://github.com/patrickfav/uber-apk-signer/releases)

```bash
java -jar apktool.jar d -o dec IlluminatiEliteGCam_v1.4_Titan2Elite.apk
cd dec && patch -p1 < ../patch/focus-ring-offset.patch && cd ..
java -jar apktool.jar b -o unsigned.apk dec
java -jar uber-apk-signer.jar --apks unsigned.apk --ks my.jks --ksAlias my --out out
```

Windows에서 사용자 폴더 이름이 한글이면 aapt2가 임시 폴더를 못 엽니다.
작업 폴더와 `-Djava.io.tmpdir`를 영문 경로로 두면 됩니다.

## 검증 방법

```bash
adb exec-out uiautomator dump /dev/tty      # capture_overlay_layout 과 viewfinder_frame 의 bounds 가 같아야 함
adb shell dumpsys media.camera | grep -A1 afRegions   # 탭 직후 HAL 에 보낸 AF 영역
```

## 이 패치가 못 고치는 것: 2x 망원

`dumpsys media.camera` 결과, 망원(ID 2)과 논리 멀티카메라(ID 3)는 `SYSTEM_CAMERA` 능력이 붙어 있어
카메라 서비스가 서드파티 앱에 ID 자체를 주지 않습니다. 시스템 속성 `ro.agui.extra_cameras_limit=yes`가 스위치로 보입니다.
어떤 GCam 포트도 우회할 수 없고, 루팅해서 속성을 바꾸는 방법만 남습니다.

| ID | 렌즈 | 레벨 | 공개 |
|---|---|---|---|
| 0 | 메인 50MP 5.59mm, RAW | LEVEL_3 | O |
| 1 | 전면 32MP 2.31mm, RAW | LEVEL_3 | O |
| 2 | 망원 6.8mm (≈2x) | FULL | SYSTEM_CAMERA |
| 3 | 논리 멀티카메라 (0+2) | LEVEL_3 | SYSTEM_CAMERA |

## 크레딧

- 포트: illuminati (Reddit r/unihertz)
- 베이스: Hasli LMC8.4 R18, Shamim SGCam, Arnova8G2, BSG, BigKaka
- 이 저장소는 표시 위치 1줄만 고쳤습니다. Google, Unihertz와 무관합니다.

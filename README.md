# VPEPlayer Android SDK — AAR 배포 저장소

네이버 클라우드 플랫폼(NCP) **VPE Android 네이티브 비디오 플레이어 SDK**의 바이너리(AAR) 배포 저장소입니다.
Jetpack Compose + Media3(ExoPlayer) 기반. 이 저장소는 **컴파일된 `aar`만** 호스팅하는 raw GitHub Maven 저장소이며 **소스는 포함하지 않습니다**.

- 좌표: `com.navercloud.vpe:player`
- 최신 버전: **2.0.1**

## 설치

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://raw.githubusercontent.com/SGRsoft-Dev/vpe2-android-native-sdk/master") }
    }
}
```
```kotlin
// app/build.gradle.kts
dependencies {
    implementation("com.navercloud.vpe:player:2.0.1")
}
```
> 인증 토큰 불필요. 공개 API 는 Media3 `@UnstableApi` 표면을 노출하므로 호출부에 `@OptIn(UnstableApi::class)` 가 필요합니다.

## 요구 사항

- **minSdk 24** · compileSdk 35 · **JDK 17** · Kotlin 2.0 · Jetpack Compose

## 빠른 시작

### 패턴 A — 간편 Composable (권장)
```kotlin
import androidx.compose.runtime.Composable
import androidx.media3.common.util.UnstableApi
import com.navercloud.vpe.player.VpePlayer

@OptIn(UnstableApi::class)
@Composable
fun Screen() {
    VpePlayer(
        accessKey = "YOUR_ACCESS_KEY",   // NCP Sub Account access key (필수)
        platform = "pub",                // "pub"(민간) / "gov"(공공)
        stage = "prod",                  // "prod" / "beta"
        options = mapOf(
            "autostart" to true,
            "aspectRatio" to "16:9",
            "playlist" to listOf(mapOf("file" to "https://.../master.m3u8")),
        ),
    )
}
```

### 패턴 B — 컨트롤러 직접 보유 (커스텀 UI)
```kotlin
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import com.navercloud.vpe.player.VpePlayerController
import com.navercloud.vpe.player.ui.VpePlayerView

@OptIn(UnstableApi::class)
@Composable
fun CustomPlayer() {
    val context = LocalContext.current
    val controller = remember {
        VpePlayerController.fromMap(
            context = context,
            options = mapOf(
                "controls" to false,
                "playlist" to listOf(mapOf("file" to "https://.../master.m3u8")),
            ),
            accessKey = "YOUR_ACCESS_KEY",
        )
    }
    val state by controller.state.collectAsStateWithLifecycle()
    DisposableEffect(controller) { onDispose { controller.destroy() } }   // 직접 보유 시 해제 필수

    VpePlayerView(controller, showBuiltinControls = false)
    Button(onClick = { controller.toggle() }) {
        Text(if (state.isPlaying) "일시정지" else "재생")
    }
}
```
> `VpePlayer(...)` Composable 은 컨트롤러를 내부 생성·자동 해제합니다. 패턴 B 처럼 `fromMap`/`fromJson` 으로 직접 만든 경우에만 `controller.destroy()` 가 필요합니다.

## 호스트 매니페스트 (권장)

전체화면 가로 전환·PiP·잠금화면 미디어 컨트롤이 매끄럽게 동작하려면:
```xml
<activity
    android:name=".MainActivity"
    android:configChanges="orientation|screenSize|smallestScreenSize|screenLayout|keyboardHidden"
    android:supportsPictureInPicture="true" />
```
- SDK 매니페스트가 `INTERNET` / `POST_NOTIFICATIONS` / `FOREGROUND_SERVICE*` 권한을 병합 제공합니다.
- 잠금화면 미디어 컨트롤은 **API 33+ 에서 `POST_NOTIFICATIONS` 런타임 권한**이 필요합니다(호스트가 요청).

## 주요 기능

- 재생: HLS(.m3u8) · MP4 · **DASH(.mpd)** — 화질 상한(ABR) 제어
- DRM: **Widevine**(`com.widevine.alpha`) — 백엔드 서명 라이선스 헤더 패스스루
- 자막: 외부 사이드카(VTT/SRT) + 매니페스트 내장, on/off·언어 로컬 영속, 시스템 접근성 자막 스타일
- 컨트롤바: 서버 주도 레이아웃 엔진 / 전체화면(가로 고정·몰입·컷아웃) / **PiP 자동 진입**
- 제스처: 좌·우 더블탭 ±10초 스킵, 길게 눌러 1.5배속
- Now Playing(MediaSession + 잠금화면 알림) · 워터마크 · 화면녹화 방지(FLAG_SECURE) · 다국어(ko·ja·en)

## 라이선스 / 문의

- 재생에는 유효한 NCP `accessKey`(결제 계정)가 필요합니다. 무효·미결제 키는 재생이 차단됩니다(E0001).
- © NAVER Cloud / SGRsoft
- 문의: **dev@sgrsoft.com**

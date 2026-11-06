# Лаунчер (com.gathin.tvhome 3.2)

Исходники: https://github.com/Gathin23/cleanyourtv, папка `launcher/`. Версия, из которой собирали: коммит в `upstream_commit.txt`.

## Что в папке
- `tvhome-3.2-redirect-launcherx.apk`: готовый APK (586 КБ). SHA-256 в `apk.sha256`.
- `config-stock_home_redirect.patch`: единственное изменение в исходниках. Строка `stock_home_redirect` указывает на `com.google.android.apps.tv.launcherx`, чтобы `HomeKeyService` перехватывал Home и возвращал новый лаунчер.
- `restore.txt`: команды отката для всех трёх приставок.

### Вариант для Mi Box 4 (приставка 3, собран 2026-11-06)
Собран специально для Xiaomi Mi Box 4 (Android 9), у которой стоковый экран `com.google.android.tvlauncher`, а `launcherx` нет. APK с `launcherx` на ней редирект не делает.
- `tvhome-3.2-redirect-tvlauncher.apk`: APK (586 КБ), `stock_home_redirect = com.google.android.tvlauncher`. SHA-256 в `apk-tvlauncher.sha256`.
- `config-stock_home_redirect-tvlauncher.patch`: тот же патч, но с `tvlauncher`.
- Исходники те же (`upstream_commit.txt`), собрано в Docker по инструкции ниже, только вместо `config-stock_home_redirect.patch` применяется `config-stock_home_redirect-tvlauncher.patch`.
- Подпись: новый debug-ключ, отличается от ключа первых двух APK. Эти APK друг на друга обновиться не могут.

## Результаты ревью кода (2026-10-31)
- В манифесте нет `INTERNET`. Разрешения: `ACCESS_NETWORK_STATE`, `ACCESS_WIFI_STATE`, `ACCESS_FINE_LOCATION` (для имени Wi-Fi, выдаётся только через `pm grant`), `WRITE_SECURE_SETTINGS`, `yandex.tv.permission.ACCESS_TV_SOURCE`.
- Нет `Runtime.exec`, `ProcessBuilder`, `DexClassLoader`, сокетов, `HttpURLConnection`, WebView.
- Службы доступности (`HomeKeyService`, `InputKeyService`) только фильтруют клавиши. Служба уведомлений `GuardRevival` уведомления не читает.
- Зависимости: `androidx.recyclerview`, `androidx.dynamicanimation`. Gradle wrapper закреплён по SHA-256.
- Подпись: debug-ключ из Docker-тома `cleanyourtv-gradle`. Обновление поверх возможно только с тем же ключом, иначе `INSTALL_FAILED_UPDATE_INCOMPATIBLE`.

## Как собрать заново
Нужен Docker. Сеть медленная, поэтому с мягкими настройками и без необязательной проверки lint:

```powershell
git clone --depth 1 https://github.com/Gathin23/cleanyourtv.git src
cd src
git apply ..\config-stock_home_redirect.patch   # или поправить config.xml вручную
$opts = '-Dorg.gradle.internal.http.connectionTimeout=180000 -Dorg.gradle.internal.http.socketTimeout=180000 -Dorg.gradle.internal.repository.max.tentatives=12'
.\tools\docker-build.ps1   # один раз, чтобы собрать образ (долго)
docker run --rm -v "${PWD}:/repo" -v "cleanyourtv-gradle:/gradle" -w /repo/launcher -e "ANDROID_USER_HOME=/gradle/.android" -e "GRADLE_OPTS=$opts" cleanyourtv-android-build sh ./gradlew --no-daemon --console=plain --max-workers=2 assembleRelease -x lintVitalAnalyzeRelease -x lintVitalReportRelease -x lintVitalRelease
```

APK: `launcher\app\build\outputs\apk\release\app-release.apk`.

## Как поставить на приставку
```bash
adb -s <ip>:5555 install -r tvhome-3.2-redirect-launcherx.apk      # Google TV, launcherx (приставки 1 и 2)
adb -s <ip>:5555 install -r tvhome-3.2-redirect-tvlauncher.apk     # Mi Box 4, tvlauncher (приставка 3)
adb -s <ip>:5555 shell 'settings put secure enabled_accessibility_services com.gathin.tvhome/.HomeKeyService; settings put secure accessibility_enabled 1'
```
Если на приставке уже стоит лаунчер, подписанный другим ключом, сначала `adb -s <ip>:5555 shell pm uninstall --user 0 com.gathin.tvhome`.
Если на приставке уже есть другие службы доступности, их нельзя затирать: список дописывается через `:` (см. `enable_service` в `launcher/build-and-install.sh` репо). После каждой переустановки APK службу нужно включить снова.

## Идеи
- Выдать `WRITE_SECURE_SETTINGS` (`pm grant`) и включить `restore_key_services`, чтобы служба сама возвращалась после принудительной остановки.
- Отключить `launcherx`: даёт больше экономии, но в репо описан чёрный Home после холодной перезагрузки (`docs/devices/acer-r4g.md`). Только с проверкой у экрана.

## Ключ подписи
APK подписан debug-ключом. Сам ключ (`debug.keystore`) хранится только локально и в репозиторий не выкладывается (см. `.gitignore`). Обновление поверх установленного лаунчера возможно только APK, подписанным тем же ключом, иначе `INSTALL_FAILED_UPDATE_INCOMPATIBLE`.

## Лицензия и происхождение
Лаунчер собран из проекта [Gathin23/cleanyourtv](https://github.com/Gathin23/cleanyourtv) (лицензия MIT) с единственным изменением из `config-stock_home_redirect.patch`. Все права на лаунчер принадлежат его автору.

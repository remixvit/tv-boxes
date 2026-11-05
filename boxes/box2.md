# Приставка 2: Xiaomi MiTV-AFKR0 («jaws»)

- IP: 192.168.1.102, ADB-порт 5555
- Android 14 (SDK 34), Google TV
- Сборка: `Xiaomi/jaws/jaws:14/UKG3.250107.001/V816.0.26.4.17.UZFAABX`
- RAM 2 ГБ (свободно около 480 МБ до чистки), накопитель 4,5 ГБ под данные: было занято 78%, после чистки кэша 69%
- Родной главный экран: Google TV Launcher (`com.google.android.apps.tv.launcherx`)
- Пакетов: 153, до нас отключено 34 (см. `backups/box2_disabled_before.txt`)

## Отключено нами (`pm disable-user --user 0`)
- Телеметрия и реклама: `com.miui.tv.analytics`, `com.xiaomi.statistic`, `com.android.adservices.api`, `com.android.ondevicepersonalization.services`, `com.android.federatedcompute.services`
- Контент Xiaomi: `com.mitv.tvhome.mitvplus`, `com.mitv.tvhome.michannel`, `com.mitv.tvhome.oemtab`, `com.xiaomi.mimusic2`
- Приложения: `com.apple.atve.androidtv.appletv`, `ru.rt.video.app.tv` (Ростелеком), `com.yandex.browser.tv`, `com.iqiyi.i18n.tv`, `ru.more.play`, `ru.mts.mtstv`, `com.google.android.play.games`, `com.google.android.youtube.tvmusic`, `com.amazon.amazonvideo.livingroom`
- Кэш приложений очищен командой `pm trim-caches 4G`.

## Оставлено
YouTube, Кинопоиск (`ru.kinopoisk.tv`), VK Видео (`com.vk.tv`), HDRezka (`com.falcofemoralis.hdrezkaapp`), SASFlix (`com.podsas.sasflix.tv`), VLC, Send Files to TV (`com.yablio.sendfilestotv`), Google TV (`com.google.android.videos`), Netflix.

## Лаунчер
- `com.gathin.tvhome` 3.2 установлен, служба `HomeKeyService` включена, `stock_home_redirect = launcherx`, `launcherx` включён.
- Первая проверка Home не засчитана (приставка была в заставке), повторная прошла: Home из Настроек открывает новый лаунчер.
- До установки службы `enabled_accessibility_services` был `null`.

## Можно сделать позже
- Удалить данные отключённых приложений (`pm clear`), чтобы освободить место.
- Поставить ivi и Okko из Google Play.

## Откат
```bash
adb -s 192.168.1.102:5555 shell settings put secure enabled_accessibility_services ""
adb -s 192.168.1.102:5555 shell pm uninstall --user 0 com.gathin.tvhome
```

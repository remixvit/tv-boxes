# Приставка 1: Xiaomi MiTV-AFMU0 («twilight»)

- IP: 192.168.1.53, ADB-порт 5555
- Android 14 (SDK 34), Google TV, патч безопасности 2026-06-01
- Сборка: `Xiaomi/twilight/twilight:14/UKG3.250826.001/V816.0.11.0.UZFAABX`
- RAM 2 ГБ (свободно около 700 МБ до чистки), накопитель около 27 ГБ, занято 12%
- Родной главный экран: Google TV Launcher (`com.google.android.apps.tv.launcherx`)
- Пакетов: 151

## Отключено до нас
- `com.netflix.ninja` и `com.amazon.amazonvideo.livingroom` (см. `backups/box1_disabled_before.txt`)

## Отключено нами (`pm disable-user --user 0`)
- Телеметрия и реклама: `com.miui.tv.analytics`, `com.xiaomi.statistic`, `com.android.adservices.api`, `com.android.ondevicepersonalization.services`, `com.android.federatedcompute.services`
- Контент Xiaomi: `com.mitv.tvhome.mitvplus`, `com.mitv.tvhome.michannel`, `com.mitv.tvhome.oemtab`, `com.xiaomi.mimusic2`
- Приложения: `com.iqiyi.i18n.tv`, `ru.rutube.app`, `com.yandex.browser.tv`, `com.google.android.play.games`, `com.google.android.youtube.tvmusic`

## Оставлено
YouTube, VK Видео (`com.vk.tv`), VLC, голосовой пульт (`katniss`, `tv.remote.service`), обновления (`xiaomi.mitv.updateservice`), `gms`, `vending`, `droidlogic.*`, Google TV Launcher (`launcherx`). PatchWall (`mitv.tvhome.atv`) отключён позже, см. «Доочистка».

Не трогали: `com.xiaomi.mitv.smartshare`, `com.xiaomi.mi_connect_service`, `com.xiaomi.wfdsinkhelperservice` (Miracast и трансляция). Галерея, видеоплеер и медиа-проводник Xiaomi отключены позже, см. «Доочистка».

## Доочистка 2026-11-06 (PatchWall и лишнее)
Отключено (`pm disable-user --user 0`): `com.mitv.tvhome.atv` (PatchWall), `com.mitv.videoplayer`, `com.mitv.gallery`, `com.xiaomi.mitv.mediaexplorer`, `com.xiaomi.android.tvsetup.partnercustomizer`, `com.xm.webcontent`, `com.google.android.feedback`, `com.android.printspooler`, `com.google.android.marvin.talkback`, `com.google.android.syncadapters.calendar`.
Оставлены Miracast-службы (`smartshare`, `mi_connect_service`, `wfdsinkhelperservice`): плитка Miracast есть на главном экране. После отключения проверены приложения и Home: YouTube и VK Видео запускаются, Home возвращает на новый лаунчер.

## Лаунчер
- `com.gathin.tvhome` 3.2 установлен, служба `HomeKeyService` включена, `stock_home_redirect = launcherx`.
- Расход памяти по `dumpsys meminfo`: новый лаунчер 60–76 МБ PSS, `launcherx` около 94 МБ.
- До установки службы `enabled_accessibility_services` был `null`.

## Откат
```bash
adb -s 192.168.1.53:5555 shell settings put secure enabled_accessibility_services ""
adb -s 192.168.1.53:5555 shell pm uninstall --user 0 com.gathin.tvhome
```

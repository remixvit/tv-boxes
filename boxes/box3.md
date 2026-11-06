# Приставка 3: Xiaomi Mi Box 4 (`MIBOX4`, «oneday»)

Дата работ: 2026-11-06.

- IP: 192.168.1.240, ADB-порт 5555
- Android 9 (SDK 28), **не Google TV**, а обычный Android TV. Патч безопасности 2021-07-05
- Сборка: `Xiaomi/oneday/oneday:9/PI/3933:user/release-keys`
- RAM 2 ГБ (свободно около 1 ГБ до чистки), накопитель около 5 ГБ под данные: было занято 78%, после чистки 58%
- Родной главный экран: `com.google.android.tvlauncher` (не `launcherx`)
- Пакетов: 97, отключённых до нас не было (см. `backups/box3_disabled_before.txt`, файл пустой)

## Как подключились
Окно подтверждения отладки сначала не появлялось. Помогло: разбудить приставку, в «Для разработчиков» отозвать доступ к отладке по USB, выключить и включить отладку по USB, затем снова `adb connect`. Отдельной «отладки по сети» в меню нет: порт 5555 открывается вместе с отладкой по USB.

## Удалено (`pm uninstall --user 0`)
more.tv (`ru.more.play`), Rutube (`ru.rutube.app`), Spotify (`com.spotify.tv.android`), Zona (`mobi.zona.tvlite`), TV Web Browser (`com.tvwebbrowser.v22`), AIDA64 (`com.finalwire.aida64`).

Возврат: поставить из Google Play или `adb shell pm install-existing <пакет>`.

## Отключено нами (`pm disable-user --user 0`)
- Телеметрия и контент Xiaomi: `com.miui.tv.analytics`, `com.mitv.tvhome.michannel`, `com.xm.webcontent`, `tv.alphonso.alphonso_eula`
- PatchWall, оболочка Xiaomi с рекомендациями: `com.mitv.tvhome.atv`. Домашним экраном он не является, около 110 МБ ОЗУ в фоне
- Приложения: `com.google.android.play.games`, `com.google.android.youtube.tvmusic`, `com.google.android.videos`, `com.netflix.ninja`, `com.amazon.amazonvideo.livingroom`
- Стоковый лаунчер Android TV (2026-11-06, позже): `com.google.android.tvlauncher` и `com.google.android.tvrecommendations`. После этого новый лаунчер `com.gathin.tvhome` остаётся единственным обработчиком Home (кроме системного `FallbackHome`), и Home открывает его сразу. Холодная перезагрузка (`adb reboot`) после отключения проверена: приставка загрузилась сразу в новый лаунчер, без чёрного экрана, служба `HomeKeyService` осталась включённой, ADB по сети вернулся через 20 секунд. Если когда-нибудь будет чёрный экран, вернуть: `adb shell pm enable com.google.android.tvlauncher`.
- Остальное лишнее: `com.mitv.videoplayer`, `com.mitv.milinkservice`, `com.xiaomi.android.tvsetup.partnercustomizer`, `android.autoinstalls.config.xioami.mibox3`, `com.google.android.backdrop` (заставка), `com.google.android.feedback`, `com.google.android.tv.bugreportsender`, `com.google.android.marvin.talkback`, `com.android.camera2`, `com.android.printspooler`, `com.google.android.syncadapters.calendar`, `com.google.android.syncadapters.contacts`.
- Всего отключено 24 пакета. После отключения проверен запуск VK Видео, Кинопоиска, ivi, TNT Premier, VLC и YouTube, все открываются, Home возвращает на новый лаунчер.

## Оставлено
VK Видео (`com.vk.tv`), Кинопоиск (`ru.kinopoisk.tv`), ivi (`ru.ivi.client`), TNT Premier (`gpm.tnt_premier`), VLC (`org.videolan.vlc`), YouTube (`com.google.android.youtube.tv`), Play Маркет, `gms`, голосовой пульт (`katniss`, `tv.remote.service`), Cast (`mediashell`), `xiaomi.mitv.updateservice`, `droidlogic.*`, стоковый лаунчер `tvlauncher`.

## Лаунчер
- `com.gathin.tvhome` 3.2 из файла `launcher/tvhome-3.2-redirect-tvlauncher.apk`. Это отдельная сборка именно для этой приставки: в ней `stock_home_redirect = com.google.android.tvlauncher`. Сборка из репо с `launcherx` тут не сработает, потому что `launcherx` на Mi Box 4 нет.
- Служба `HomeKeyService` включена. До этого `enabled_accessibility_services` был `null`, `accessibility_enabled` = 0.
- Назначение домашним через `cmd package set-home-activity` не помогло: прошивка продолжает открывать `tvlauncher`. Работает только перехват службой доступности.
- Проверка Home из VK Видео и Кинопоиска через `adb input keyevent 3`: новый лаунчер открывается, но с задержкой около 4 секунд (сначала мелькает стоковый экран). Реальным пультом не проверяли.
- Расход памяти нового лаунчера около 30 МБ PSS.
- APK подписан новым debug-ключом, не тем, что у приставок 1 и 2. Ставили с удалением старой версии (`pm uninstall`), иначе `INSTALL_FAILED_UPDATE_INCOMPATIBLE`.

## Откат
```bash
adb -s 192.168.1.240:5555 shell settings put secure enabled_accessibility_services ""
adb -s 192.168.1.240:5555 shell pm uninstall --user 0 com.gathin.tvhome
adb -s 192.168.1.240:5555 shell pm enable <пакет>
```
Последняя команда возвращает один отключённый пакет.

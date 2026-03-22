# TeerkVPN (Android)

Минимальный пример Android‑клиента VPN на базе VpnService и Xray/VLESS (Jetpack Compose, Kotlin).

## Возможности
- Главное окно с одной кнопкой включения/выключения VPN и автовор выбора сервера с минимальным пингом из включенных.
- Экран серверов: просмотр всех узлов, включение/отключение, ручной выбор, обновление пинга.
- Экран приложений: режим «Все приложения» или выбор конкретных пакетов, которые должны идти через VPN.
- Заготовка менеджера Xray (`vpn/XrayManager.kt`) и службы `XrayVpnService` для дальнейшей интеграции с настоящим xray-core.

## Быстрый старт
1. Откройте проект в Android Studio (комpileSdk 34, JDK 17).  
2. Подмените тестовые сервера в `repo/ServerRepository.kt` своими VLESS‑узлами.  
3. Соберите/установите: `./gradlew assembleDebug`.

## Как подключить реальный xray-core
Текущее приложение использует заглушку. Чтобы получить рабочее подключение:
1. Добавьте в `app/libs` aar/so библиотеки с собранным xray-core (например, из [AndroidLibXrayLite](https://github.com/MatsuriDayo/AndroidLibXrayLite)) и подключите её в `app/build.gradle.kts` (`implementation(files("libs/your-xray.aar"))`).
2. В `vpn/XrayManager.kt` реализуйте запуск/остановку ядра:  
   - Сформируйте JSON‑конфиг VLESS (можно через зависимость `io.github.tim06.xray-configuration`).  
   - Передайте правила для приложений: `allowedApps` в `VpnService.Builder.addAllowedApplication()` или `disallowedApps` для обхода.  
   - Запустите Xray поверх TUN-интерфейса, созданного в `XrayVpnService`.
3. В `XrayVpnService` поднимите реальный туннель: создайте `Builder()`, задайте DNS/маршруты, вызовите `establish()` и передайте дескриптор в Xray.

## Замечания
- Пинг считается через TCP connect к `address:port` (timeout 1.5с). При необходимости замените на ICMP/HTTP.  
- Для отображения установленных приложений используется `QUERY_ALL_PACKAGES`; на Google Play может потребовать обоснование.  
- Уведомление службы использует системную иконку; при желании замените на собственный `@mipmap/ic_launcher`.

## Структура
- `MainActivity` / `ui/*` — Compose‑интерфейс.  
- `viewmodel/VpnViewModel` — состояние и бизнес‑логика.  
- `repo/*` — серверы, приложения, пинг.  
- `vpn/*` — слой интеграции с Xray/VpnService.

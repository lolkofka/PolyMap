# Релиз PolyMap в Google Play

Android-исходники основаны на ветке `android-pr`, коммит
`0e948fb19a341e4cca74386af2e4237a3b8dfdb5`.
Ветка подготовки релиза: `release/google-play`.

## Сборка

- Package name: `com.polymap.android`.
- Версия в исходниках: `1.1.1`, versionCode `6`.
- compileSdk / targetSdk: `36`, minSdk: `26`.
- Системный возврат переведён с `Activity.onBackPressed()` на AndroidX
  `OnBackPressedDispatcher`; перехват включается только при наличии вложенной страницы.
- В Google Play предполагается вариант `gl` (OpenGL); вариант `vulkan` не включать в тот же релиз.
- Нужны JDK 17+, Android SDK Platform 36 и Build Tools 35.0.0.

```sh
cd android
./gradlew :app:bundleGlRelease :app:testGlDebugUnitTest :app:lintVitalGlRelease
```

Результат: `app/build/outputs/bundle/glRelease/app-gl-release.aab`.
Без параметров подписи это **неподписанный** AAB для проверки сборки, его нельзя загружать в Play Console.

Для подписанного AAB задаются все четыре переменные окружения:

- `POLYMAP_UPLOAD_STORE_FILE` — абсолютный путь к upload-keystore;
- `POLYMAP_UPLOAD_STORE_PASSWORD` — пароль хранилища;
- `POLYMAP_UPLOAD_KEY_ALIAS` — alias upload-ключа;
- `POLYMAP_UPLOAD_KEY_PASSWORD` — пароль upload-ключа.

Ключи и пароли не добавляются в Git. Пароли не передаются аргументами командной строки.
Если задана только часть переменных, конфигурация сборки завершается ошибкой.
Release не использует debug-ключ.

Если приложение уже создано в Play Console, сначала проверить его package name,
последний versionCode и сертификат upload-ключа. Новый ключ не заменяет зарегистрированный автоматически.
Для нового приложения можно создать отдельный upload-ключ и включить Play App Signing.

## Описание магазина

Русские тексты находятся в `fastlane/metadata/android/ru-RU/`.
Полное описание прямо указывает, что это неофициальный порт и полноценный форк,
содержит указание на разрешение разработчика оригинала (по информации владельца публикации)
и ссылку на открытые Android-исходники релизной ветки.

## Что ещё требуется до отправки

1. Проверить доступ к аккаунту Play Console, существующую карточку приложения и статус аккаунта.
2. Подписать AAB соответствующим upload-ключом.
3. Прогнать релиз на устройствах: запуск карты, поиск аудитории, этажи, маршрут и расписание.
4. Проверить 16-КБ выравнивание всех нативных библиотек в итоговом AAB.
5. Разместить публичную политику конфиденциальности и сделать её доступной в приложении.
6. Заполнить Data safety, возрастной рейтинг, целевую аудиторию и декларацию рекламы.
7. Добавить иконку магазина, скриншоты и feature graphic.
8. Проверить необходимость закрытого тестирования в конкретном аккаунте.

## Проверенные по исходникам потоки данных

- `ReportApi.kt`: сообщение пользователя, идентификаторы объекта/маршрута, параметры маршрута,
  модель устройства, версия Android и приложения, ориентация, язык и размер экрана
  отправляются на `https://polymap.ru/api/report/...` по действию пользователя.
- `CodeGeneratorApi.kt`: создание кода приглашения отправляет начало и конец маршрута,
  текст приглашения и параметры маршрута на `https://polymap.ru/api/generate`.
- `TimetableApi.kt`: запросы расписания направляются на `https://ruz.spbstu.ru/api/v1/ruz`.
- `PolyMapView.kt`: стили/тайлы карты загружаются с OpenFreeMap, резервная подложка — OpenStreetMap.
  Эти серверы получают сетевые запросы и IP-адрес; точные правила обработки данных проверяются у операторов.
- История поиска, избранное и настройки хранятся в SharedPreferences приложения.
- В исходном манифесте нет разрешений GPS, камеры, микрофона или контактов.
  Рекламный и аналитический SDK в явных зависимостях Gradle не обнаружен.

Это аудит исходников, а не подтверждение серверных сроков хранения или полного поведения сторонних SDK.
Для политики и Data safety ещё нужны контакт разработчика, порядок удаления и сроки хранения
данных на сервере, а также проверка поведения MapLibre и поставщиков карт.

## Официальные требования

- Target API: https://developer.android.com/google/play/requirements/target-sdk
- App Bundles: https://developer.android.com/guide/app-bundle
- Подпись: https://developer.android.com/studio/publish/app-signing
- 16 КБ: https://developer.android.com/guide/practices/page-sizes
- Политика данных: https://support.google.com/googleplay/android-developer/answer/10144311
- Тестирование личных аккаунтов: https://support.google.com/googleplay/android-developer/answer/14151465

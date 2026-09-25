# Чеклист публикации Tutor в Google Play

Состояние кода на момент анализа (Android-репо / MVP): debug-сборка работает, `versionName=0.1.0`, `USE_MOCK_AUTH=true` по умолчанию, release без signing/minify, иконка vector-only, mock OAuth.

Связанный документ: [../data-safety/README.md](../data-safety/README.md)

---

## A. Обязательно до первого AAB

### 1. Аккаунт и пакет
- [ ] Google Play Console (developer account, разовый взнос)
- [ ] Зафиксировать финальный `applicationId` (сейчас `com.tutor.english`) — после первой публикации менять нельзя
- [ ] Создать приложение в Console, выбрать тип (app), free/paid

### 2. Подпись и сборка release
- [ ] Создать upload keystore (не коммитить; хранить backup)
- [ ] Настроить `signingConfigs.release` в `app/build.gradle.kts`
- [ ] Собрать **AAB**: `./gradlew :app:bundleRelease`
- [ ] Включить Play App Signing
- [ ] Поднять `versionCode` / осмысленный `versionName` (не оставлять `0.1.0` без решения)

### 3. Выключить mock / debug-хвосты
Сейчас в проекте:
- `USE_MOCK_AUTH=true`, `USE_MOCK_PRONUNCIATION=true` по умолчанию
- На Login — текст про test account / mock OAuth
- Соцлогин — stub (`socialSignInStub`), не реальные Yandex/Google/Apple
- API-ключи через `BuildConfig` из `local.properties` (не класть секреты в git; для CI — secrets)

Сделать:
- [ ] Release: `USE_MOCK_AUTH=false`, `USE_MOCK_PRONUNCIATION=false` (или remote config)
- [ ] Убрать debug-подсказки с LoginScreen из release (`BuildConfig.DEBUG`)
- [ ] Решить: реальный auth backend **или** оставить email-only без fake social кнопок в store-сборке
- [ ] Не логировать токены/ключи (OkHttp logging только DEBUG — уже так)

### 4. Безопасность release
- [ ] `isMinifyEnabled = true` (+ R8) для release, дописать `proguard-rules.pro` (Retrofit, Kotlinx Serialization, Hilt, Azure Speech, Room)
- [ ] Проверить crash после minify на устройстве
- [ ] `allowBackup`: сейчас `true` — решить exclude чувствительных данных или `false`
- [ ] API keys: предпочтительно не в BuildConfig APK (прокси на softverno.ru / remote config)

### 5. Store listing assets
- [ ] Иконка 512×512 (сейчас в res в основном adaptive XML — для Play нужна растровая high-res)
- [ ] Feature graphic 1024×500
- [ ] Скриншоты phone (минимум 2), желательно 7" / tablet если поддерживаете
- [ ] Короткое описание (80), полное (4000)
- [ ] Категория: Education
- [ ] Контакт разработчика, email поддержки

### 6. Политики и URL
В коде уже ссылки на:
- `https://softverno.ru/privacy`
- `https://softverno.ru/terms`

Нужно:
- [ ] Страницы **реально существуют** и актуальны (AI processors: DeepSeek, Groq, Azure, Speechace, Google Play Billing)
- [ ] Data Safety в Console заполнить по черновику в [`../data-safety/`](../data-safety/)
- [ ] Account deletion: в аппе есть DeleteAccount 14d — описать в политике и в Console «Data deletion»

### 7. Разрешения и декларации Console
- [ ] `RECORD_AUDIO` — обосновать в форме Sensitive permissions / Data safety (voice)
- [ ] Если нет рекламы — Advertising ID = No
- [ ] Content rating questionnaire (IARC)
- [ ] Target audience / Children: если не Kids — явно не Families
- [ ] News / Health / Finance — обычно N/A

### 8. Подписки (Play Billing)
В коде есть Soft Paywall + BillingClient:
- [ ] Создать products/subscriptions в Play Console
- [ ] Сопоставить SKU с кодом
- [ ] Лицензионное тестирование (license testers)
- [ ] Base plan / offers, trial 7 дней — согласовать с UX-копирайтом

### 9. Качество перед review
- [ ] Прогнать critical path: onboarding → chat mic → settings → paywall → logout/delete
- [ ] Нет крашей на API 26+ (minSdk 26), targetSdk 35 — ок
- [ ] RTL smoke (ar)
- [ ] Offline / нет ключа Azure — fallback TTS
- [ ] Pre-launch report (после загрузки AAB во внутренний тест)

---

## B. Рекомендуемый порядок работ

1. **Юридическое** — privacy/terms на softverno.ru + Data Safety  
2. **Продукт release** — выключить mock, убрать debug UI, решить auth  
3. **Техническое** — signing, minify, AAB  
4. **Store** — тексты, иконка, скрины  
5. **Billing** — продукты + тест покупки  
6. **Internal testing** → Closed → Production  

---

## C. Что уже относительно готово

- `targetSdk = 35`, `minSdk = 26`
- HTTPS only (`usesCleartextTraffic=false`)
- Нет лишних dangerous permissions
- Terms/Privacy deep links в регистрации
- Delete account + Data Safety черновик
- Play Billing зависимость уже в проекте

---

## D. Блокеры «нельзя в Production as-is»

1. Нет release signing config  
2. Mock auth / test account UI  
3. Social login stub (если кнопки видны ревьюеру — риск отклонения как misleading)  
4. Нет store graphics (512 icon, feature graphic, screenshots)  
5. Privacy/Terms должны быть живыми URL  
6. Minify выключен (не блокер Play, но риск для безопасности/размера)  
7. Секреты в BuildConfig APK легко извлекаются  

---

## Store listing — артефакты (чеклист файлов)

Положить готовые файлы рядом (не коммитить бинарники без нужды) или хранить в Drive / Console:

| Артефакт | Спека |
|---|---|
| App icon | 512×512 PNG |
| Feature graphic | 1024×500 |
| Phone screenshots | ≥2 |
| Short description | ≤80 chars |
| Full description | ≤4000 chars |
| Support email | contact разработчика |

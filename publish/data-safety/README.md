# Google Play Data Safety — черновик (Tutor / softverno)

Источник анализа: `app/build.gradle.kts`, `gradle/libs.versions.toml`, `app/src/main/AndroidManifest.xml` (Android-репозиторий приложения)  
Дата черновика: 2026-09-23  
Статус: **черновик для юриста / публикатора**, не финальная декларация.

Связанный чеклист: [../play-store/README.md](../play-store/README.md)

---

## 1. Разрешения из AndroidManifest

| Permission | Назначение в приложении |
|---|---|
| `INTERNET` | API: DeepSeek (чат), Groq Whisper (STT), Azure Speech (TTS/голоса), Speechace (произношение, опционально), Play Billing, открытие softverno.ru |
| `RECORD_AUDIO` | Практика речи: запись микрофона → STT / оценка произношения |
| `ACCESS_NETWORK_STATE` | Проверка сети для fallback Azure → системный TTS |

**Не заявлены (и не используются в манифесте):** `CAMERA`, `ACCESS_FINE/COARSE_LOCATION`, `READ/WRITE_EXTERNAL_STORAGE`, `READ_MEDIA_*`, `POST_NOTIFICATIONS` (для Android 13+ — если включите push, добавить), `AD_ID` / Advertising ID, `BLUETOOTH`, контакты, SMS, телефон.

**Примечание:** `Play Billing` / Google Play Services могут транзитивно запрашивать свои разрешения через merged manifest — при релизе сверить `merged_manifest`. Billing обычно не требует объявлять AD_ID, если нет рекламы.

---

## 2. Сторонние библиотеки (runtime)

### Google / AndroidX (UI, платформа)
| Артефакт | Версия / BOM | Данные / сеть |
|---|---|---|
| androidx.core:core-ktx | 1.15.0 | Нет отправки ПДн |
| androidx.appcompat:appcompat | 1.7.0 | Нет |
| androidx.lifecycle:* | 2.8.7 | Нет |
| androidx.activity:activity-compose | 1.9.3 | Нет |
| androidx.compose.* (BOM) | 2024.12.01 | Нет |
| androidx.navigation:navigation-compose | 2.8.5 | Нет |
| androidx.datastore:datastore-preferences | 1.1.1 | Локальные настройки |
| androidx.security:security-crypto | 1.1.0-alpha06 | Локальное шифрование токенов (Keystore) |
| androidx.room:* | 2.6.1 | Локальная БД (чат/прогресс) |
| androidx.hilt:hilt-navigation-compose | 1.2.0 | DI |

### Google Play / DI
| Артефакт | Версия | Данные / сеть |
|---|---|---|
| com.android.billingclient:billing-ktx | 7.1.1 | Покупки через Google Play; Google обрабатывает платёжные данные |
| com.google.dagger:hilt-android (+ compiler KSP) | 2.53.1 | Нет отправки ПДн |

### Сеть / сериализация
| Артефакт | Версия | Данные / сеть |
|---|---|---|
| com.squareup.retrofit2:retrofit | 2.11.0 | HTTP-клиент |
| com.squareup.okhttp3:okhttp + logging-interceptor | 4.12.0 | HTTP; logging не должен писать токены в release |
| org.jetbrains.kotlinx:kotlinx-serialization-json | 1.7.3 | Нет |
| com.jakewharton.retrofit:retrofit2-kotlinx-serialization-converter | 1.0.0 | Нет |
| org.jetbrains.kotlinx:kotlinx-coroutines-android | 1.9.0 | Нет |

### AI / Speech (облачные сервисы)
| Артефакт / сервис | Версия / endpoint | Какие данные уходят |
|---|---|---|
| com.microsoft.cognitiveservices.speech:client-sdk | 1.40.0 → Azure | Текст TTS, аудио синтеза; ключ/регион из BuildConfig |
| Groq Whisper (через Retrofit/OkHttp) | api.groq.com | Аудиозапись пользователя (STT) |
| DeepSeek (через Retrofit) | api.deepseek.com | Текст чата / промпты |
| Speechace (через Retrofit, опционально) | api.speechace.com | Аудио + reference text для произношения |

### UI прочее
| Артефакт | Версия | Данные |
|---|---|---|
| com.github.PhilJay:MPAndroidChart | v3.1.0 | Только локальные графики прогресса |

### Только тесты (не в production APK)
JUnit, AndroidX Test, Espresso, Compose UI Test — **не указывать** в Data Safety как сборщики данных.

---

## 3. Черновик ответов формы Data Safety

### Overview
| Вопрос формы | Предлагаемый ответ |
|---|---|
| Does your app collect or share any of the required user data types? | **Yes** |
| Is all of the user data collected by your app encrypted in transit? | **Yes** (HTTPS / `usesCleartextTraffic="false"`) |
| Do you provide a way for users to request that their data is deleted? | **Yes** (Delete account + grace period 14 days в Settings; указать URL политики softverno.ru) |

### Data collected — по типам Google

#### Personal info
| Тип | Collect? | Share? | Purpose | Required / Optional | Ephemeral? |
|---|---|---|---|---|---|
| Name | Yes (display name) | No* | App functionality, Account management | Optional | No |
| Email address | Yes | No* | Account management, App functionality | Optional (guest возможен) | No |
| User IDs | Yes | No* | App functionality, Account management | Required for logged-in | No |
| Phone number | Yes (если регистрация по телефону) | No* | Account management | Optional | No |
| Address / Race / Political / Sexual / Other | **No** | — | — | — | — |

\*Share = «передача третьим лицам для их собственных целей». Передача процессорам (Azure/Groq/DeepSeek) обычно оформляется как **Data processors / service providers**, не как «share» для рекламы — в форме: **Not shared** / или указать service providers по политике Google.

#### Financial info
| Тип | Collect? | Share? | Notes |
|---|---|---|---|
| Purchase history | Yes (через Play Billing entitlement) | With Google Play | Purpose: App functionality |
| Credit card / Bank | **No** (обрабатывает Google Play) | — | Не заявлять как сбор приложением |

#### Health / Messages / Photos / Files / Calendar / Contacts
**Не собираем** (если не добавите экспорт файлов наружу).

#### Audio (Voice or sound recordings)
| | |
|---|---|
| Collect? | **Yes** |
| Share? | **Yes — service providers** (Groq STT; Speechace/Azure при включении) |
| Purpose | App functionality (language practice, pronunciation) |
| Required? | Optional (можно текстом) |
| Ephemeral? | Можно отметить **ephemeral** для потокового STT, если запись не хранится на сервере дольше обработки; локальный кэш TTS — не user voice |

#### App activity
| Тип | Collect? | Purpose |
|---|---|---|
| App interactions | Yes (локально + лёгкая analytics в logcat facade) | App functionality, Analytics |
| In-app search history | No | — |
| Other user-generated content | Yes (чат с репетитором) | App functionality → может уходить в DeepSeek |
| Other actions | Progress / mistakes локально (Room) | App functionality |

#### App info and performance
| Тип | Collect? | |
|---|---|---|
| Crash logs | No (пока нет Crashlytics/Sentry) | Если добавите — обновить |
| Diagnostics | No | |
| Other performance | No | |

#### Device or other IDs
| Тип | Collect? | |
|---|---|---|
| Device or other IDs | **Possibly** через Play Billing / Google Play Services | Не используем Advertising ID; **не продаём данные**, рекламы нет |
| AD_ID | **No** (нет рекламного SDK) | В форме: Advertising / marketing — No |

### Purposes checklist (для отмеченных типов)
- **App functionality** — да (основное)
- **Analytics** — да (лёгкий Tracker; уточнить, уходит ли off-device; сейчас — в основном log)
- **Developer communications** — нет (пока)
- **Advertising / marketing** — **нет**
- **Fraud prevention / Security** — да (токены, сессия 7 дней)
- **Account management** — да
- **Personalization** — да (настройки голоса/языка; опциональный toggle)

### Data sharing / sale
| Вопрос | Ответ |
|---|---|
| Data sold? | **No** |
| Data shared for advertising? | **No** |
| Independent third parties with own purposes | Обычно **No**; облачные AI — **service providers** по вашему DPA |

### Security practices
| | |
|---|---|
| Encryption in transit | Yes (TLS) |
| Encryption at rest (sensitive) | Yes for auth tokens (`EncryptedSharedPreferences` + Keystore); Room/DataStore — стандартное локальное хранение |
| Users can request deletion | Yes |
| Committed to Play Families Policy | Только если в Family; иначе N/A |

### Preview / Data safety labels (кратко для карточки)
- **Data collected:** Email, Name, User ID, Phone (optional), Voice recordings, App activity / chat content, Purchase history  
- **Data shared:** Voice/text with speech & AI providers (Groq, Azure, DeepSeek; Speechace if enabled); purchases with Google  
- **Security:** Data encrypted in transit; account deletion available  
- **Not collected:** Precise location, contacts, photos, advertising ID  

---

## 4. Service providers (для Privacy Policy, не всегда отдельная строка формы)

Указать в политике softverno.ru:
1. **Google Play** — биллинг  
2. **DeepSeek** — генерация ответов репетитора  
3. **Groq** — распознавание речи  
4. **Microsoft Azure Cognitive Services** — синтез речи / каталог голосов  
5. **Speechace** — оценка произношения (если ключ задан)  

---

## 5. Риски / что поправить до публикации

1. `android:allowBackup="true"` — бэкап может включать чувствительные prefs; для токенов уже EncryptedSharedPreferences, но Room может попасть в backup — рассмотреть `allowBackup="false"` или exclude.  
2. Нет `POST_NOTIFICATIONS` — если reminders/push включите, добавить permission + Data Safety «Notifications».  
3. Analytics сейчас без Firebase — не заявлять Firebase Analytics, пока не подключите.  
4. Mock OAuth (Yandex/Google/Apple) — когда подключете реальные SDK, обновить Data Safety (account info от IdP).  
5. Сверить **merged manifest** после `./gradlew :app:processReleaseManifest` на скрытые permissions от Billing/Speech SDK.  

---

## 6. Готовые формулировки (RU) для политики / формы

> Приложение Tutor собирает данные аккаунта (email, имя, при необходимости телефон), записи голоса для практики языка и тексты диалогов с ИИ-репетитором. Данные передаются обработчикам: DeepSeek (чат), Groq (распознавание речи), Microsoft Azure (синтез речи), при включении — Speechace (произношение), а также Google Play (подписки). Мы не продаём данные и не используем их для рекламы. Передача защищена HTTPS; токены входа хранятся в зашифрованном виде на устройстве. Удаление аккаунта доступно в настройках (льготный период 14 дней).

---

*Конец черновика.*

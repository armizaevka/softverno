# Softverno

Публикационные материалы Softverno / Tutor для Google Play.

Это **не** исходники Android-приложения. Здесь — чеклист релиза, черновик Data Safety и структура артефактов store listing. Политика и условия: [softverno.ru/privacy](https://softverno.ru/privacy), [softverno.ru/terms](https://softverno.ru/terms).

## Как публиковать

1. Пройти чеклист в [`publish/play-store/`](publish/play-store/README.md).
2. Заполнить Data Safety в Play Console по [`publish/data-safety/`](publish/data-safety/README.md).
3. Собрать AAB и store assets в отдельном Android-репозитории / CI (не в этом репо, пока сюда не перенесён monorepo приложения).
4. Загрузить во внутренний тест → closed → production.

Mac не требуется для работы с этими документами. APK/AAB сюда не коммитятся.

## Структура

```
publish/
  README.md
  play-store/     # чеклист публикации и store listing
  data-safety/    # черновик формы Data Safety
```

# 🚀 GitHub Action: Автоматичний деплой на FTP з нотифікаціями в Slack

Цей GitHub Action дозволяє автоматизувати деплой змін з репозиторію у **WordPress тему** через FTP.
Додатково налаштовані повідомлення у **Slack** про статус деплою (успіх / помилка).

---

## 1. Умова запуску

Action спрацьовує автоматично при пуші в гілки:

* `master`
* `main`

```yaml
on:
  push:
    branches:
      - master
      - main
```

---

## 2. Виконання задачі

### 2.1. Середовище

Запуск виконується на **Ubuntu**:

```yaml
runs-on: ubuntu-latest
```

### 2.2. Кроки виконання

#### 🔹 Крок 1. Checkout коду

Завантаження коду з репозиторію:

```yaml
- name: 📥 Checkout code
  uses: actions/checkout@v4
```

---

#### 🔹 Крок 2. Деплой файлів на FTP

Використовується [SamKirkland/FTP-Deploy-Action](https://github.com/SamKirkland/FTP-Deploy-Action).
Деплой відбувається з локальної директорії `./wp-content/themes/` у відповідну папку на сервері `/wp-content/themes/`.

```yaml
- name: 📤 Deploy "themes" to FTP
  uses: SamKirkland/FTP-Deploy-Action@v4.3.5
  with:
    server: ${{ secrets.FTP_SERVER }}
    username: ${{ secrets.FTP_USERNAME }}
    password: ${{ secrets.FTP_PASSWORD }}
    local-dir: ./wp-content/themes/
    server-dir: /wp-content/themes/
    protocol: ftp
    exclude: |
      .git*
      .github*
      *.md
      **/node_modules/**
      **/vendor/**
```

---

#### 🔹 Крок 3. Нотифікації у Slack

Для повідомлень використовується [rtCamp/action-slack-notify](https://github.com/rtCamp/action-slack-notify).

##### ✅ Успішний деплой

```yaml
- name: ✅ Notify Slack success
  if: success()
  uses: rtCamp/action-slack-notify@v2
  env:
    SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
    SLACK_USERNAME: "GitHub Actions"
    SLACK_ICON_EMOJI: ":white_check_mark:"
    SLACK_COLOR: ${{ job.status }}
    SLACK_MESSAGE: |
      ✅ *Deploy finished successfully*
      Repository: `${{ github.repository }}`
      Branch: `${{ github.ref_name }}`
```

##### ❌ Невдалий деплой

```yaml
- name: ❌ Notify Slack failure
  if: failure()
  uses: rtCamp/action-slack-notify@v2
  env:
    SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
    SLACK_USERNAME: "GitHub Actions"
    SLACK_ICON_EMOJI: ":x:"
    SLACK_COLOR: ${{ job.status }}
    SLACK_MESSAGE: |
      ❌ *Deploy failed*
      Repository: `${{ github.repository }}`
      Branch: `${{ github.ref_name }}`
```

---

## 3. Додавання GitHub Secrets

Щоб захистити чутливі дані (логін, пароль, вебхуки), вони зберігаються у **GitHub Secrets**.

### Кроки:

1. Відкрити репозиторій у GitHub.
2. Перейти у меню **Settings**.
3. Вибрати розділ **Secrets and variables → Actions**.
4. Натиснути **New repository secret**.
5. Додати такі секрети:

   * `FTP_SERVER` – адреса FTP-сервера
   * `FTP_USERNAME` – логін користувача
   * `FTP_PASSWORD` – пароль користувача
   * `SLACK_WEBHOOK_URL` – URL вебхука для Slack

---

## 4. Повний приклад файлу `.github/workflows/deploy.yml`

```yaml
name: 🚀 Deploy to FTP on branch push

on:
  push:
    branches:
      - master
      - main

jobs:
  ftp-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: 📥 Checkout code
        uses: actions/checkout@v4

      - name: 📤 Deploy "themes" to FTP
        uses: SamKirkland/FTP-Deploy-Action@v4.3.5
        with:
          server: ${{ secrets.FTP_SERVER }}
          username: ${{ secrets.FTP_USERNAME }}
          password: ${{ secrets.FTP_PASSWORD }}
          local-dir: ./wp-content/themes/
          server-dir: /wp-content/themes/
          protocol: ftp
          exclude: |
            .git*
            .github*
            *.md
            **/node_modules/**
            **/vendor/**

      - name: ✅ Notify Slack success
        if: success()
        uses: rtCamp/action-slack-notify@v2
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_USERNAME: "GitHub Actions"
          SLACK_ICON_EMOJI: ":white_check_mark:"
          SLACK_COLOR: ${{ job.status }}
          SLACK_MESSAGE: |
            ✅ *Deploy finished successfully*
            Repository: `${{ github.repository }}`
            Branch: `${{ github.ref_name }}`

      - name: ❌ Notify Slack failure
        if: failure()
        uses: rtCamp/action-slack-notify@v2
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
          SLACK_USERNAME: "GitHub Actions"
          SLACK_ICON_EMOJI: ":x:"
          SLACK_COLOR: ${{ job.status }}
          SLACK_MESSAGE: |
            ❌ *Deploy failed*
            Repository: `${{ github.repository }}`
            Branch: `${{ github.ref_name }}`
```

---

## 5. Що потрібно зробити розробнику

1. Створити файл `.github/workflows/deploy.yml` у своєму репозиторії.
2. Додати у **GitHub Secrets**:

   * `FTP_SERVER`
   * `FTP_USERNAME`
   * `FTP_PASSWORD`
   * `SLACK_WEBHOOK_URL`
3. Закомітити файл у репозиторій.
4. Перевірити роботу: зробити пуш у `master` або `main`.

## Скриншотные тесты (playwright)

### Инициализация

Первым делом перед развертывании скриншотных тестов, нужно устанвоить dev зависимость

```bash
npm i -D @playwright/test
```

Следующим шагом нужно развернуть `playwright` с помощью команды

```bash
npm init playwright@latest 
```

Первым делом у нас спросят, где будут храниться наши тесты, советую указать `./src/__tests__`, далее читаем что нужно и ставим Y или N

Следующем шагом у нас создадутся папка `__tests__` в src, где будет один шаблонный тест и папка `test_examples`, которую можно сразу удалить. Так же будет конфиг настроек `playwright.config.ts` в корне нашего проекта

### Настройка скриншотных тестов

Первым делом, советую настроить экраны, на которых будут `playwright` будет делать скриншоты, для этого переходим в корне нашего проекта `playwright.config.ts` и видим вот такое вот конфиг

> :warning: **Важно:** Версию `playwright` я использую 1.39, время от времени возможно кфг может менятся, но суть будет той же

<details>
  <summary>Конфиг playwright.config.ts</summary>

  ```javascript
import { defineConfig, devices } from "@playwright/test";

/**
 * Read environment variables from file.
 * https://github.com/motdotla/dotenv
 */
// require('dotenv').config();

/**
 * See https://playwright.dev/docs/test-configuration.
 */
export default defineConfig({
  testDir: "./src/__tests__",
  /* Run tests in files in parallel */
  fullyParallel: true,
  /* Fail the build on CI if you accidentally left test.only in the source code. */
  forbidOnly: !!process.env.CI,
  /* Retry on CI only */
  retries: process.env.CI ? 2 : 0,
  /* Opt out of parallel tests on CI. */
  workers: process.env.CI ? 1 : undefined,
  /* Reporter to use. See https://playwright.dev/docs/test-reporters */
  reporter: "html",
  /* Shared settings for all the projects below. See https://playwright.dev/docs/api/class-testoptions. */
  expect: {
    toHaveScreenshot: { maxDiffPixels: 100 },
  },
  use: {
    /* Base URL to use in actions like `await page.goto('/')`. */
    // baseURL: 'http://127.0.0.1:3000',

    /* Collect trace when retrying the failed test. See https://playwright.dev/docs/trace-viewer */
    trace: "on-first-retry",
  },

  /* Configure projects for major browsers */
  projects: [
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
    },

    {
      name: "firefox",
      use: { ...devices["Desktop Firefox"] },
    },

    {
      name: "webkit",
      use: { ...devices["Desktop Safari"] },
    },

    /* Test against mobile viewports. */
    // {
    //   name: 'Mobile Chrome',
    //   use: { ...devices['Pixel 5'] },
    // },
    // {
    //   name: 'Mobile Safari',
    //   use: { ...devices['iPhone 12'] },
    // },

    /* Test against branded browsers. */
    // {
    //   name: 'Microsoft Edge',
    //   use: { ...devices['Desktop Edge'], channel: 'msedge' },
    // },
    // {
    //   name: 'Google Chrome',
    //   use: { ...devices['Desktop Chrome'], channel: 'chrome' },
    // },
  ],

  /* Run your local dev server before starting the tests */
  // webServer: {
  //   command: 'npm run start',
  //   url: 'http://127.0.0.1:3000',
  //   reuseExistingServer: !process.env.CI,
  // },
});

```
</details>

В конфиге видим массив объектов `projects`, сейчас там стоят дефолтные настройки, если мы хотим добавить какое новое значение, то тогда нужно добавить объект вот такого формата

```javascript
{
  name: "chromium:IphoneSe", // Название не должны быт схожи
  use: { ...devices["iPhone SE"] },
},
```

В devices очень много разных форматов, выберайте которые вам нужны
Так же хочу обратить внимание на настройку `testDir`, этот параметр отвечает за путь к вашим тестам

Так же в `pacjage.json` надо добавить 2 скрипта
```javascript
  "test:screen": "playwright test", // Запуск тестов
  "screen:update": "playwright test  --update-snapshots" // Обновление сриншотов
```

### Написание тестов

Теперь приступаем к самому интересному, начинаем писать тесты, будем делать все по пунктам

1. Переходим в папку `src/__tests__` и создаем папку, которая будет отображать название страницы, которую мы тестируем

<details>
  <summary>Часто используемые методы в Playwright</summary>

  ## Навигация
- `page.goto('url')`: Переход на указанный URL.
- `page.reload()`: Перезагрузка текущей страницы.

## Поиск элементов
- `page.$('selector')`: Находит первый элемент на странице, соответствующий селектору.
- `page.$$('selector')`: Находит все элементы на странице, соответствующие селектору.
- `page.$eval('selector', func)`: Выполняет функцию `func` на элементе, найденном с помощью селектора.

## Взаимодействие с элементами
- `page.click('selector')`: Клик по элементу.
- `page.dblclick('selector')`: Двойной клик по элементу.
- `page.fill('selector', 'text')`: Заполнение текстового поля.
- `page.type('selector', 'text')`: Печать текста в текстовом поле.
- `page.check('selector')`: Установка флажка (чекбокса).
- `page.uncheck('selector')`: Снятие флажка (чекбокса).
- `page.selectOption('selector', 'value')`: Выбор значения из выпадающего списка.

## Работа с JavaScript
- `page.evaluate(() => {})`: Выполнение произвольного JS-кода на странице.

## Работа со скриншотами и PDF
- `page.screenshot()`: Создание скриншота страницы.
- `page.pdf()`: Сохранение страницы в формате PDF (только в headless режиме).

## Ожидание элементов и событий
- `page.waitForTimeout(ms)`: Ожидание заданного количества миллисекунд.
- `page.waitForSelector('selector')`: Ожидание появления элемента на странице.
- `page.waitForFunction(func)`: Ожидание выполнения JS-функции.

## Работа с контекстами браузера и вкладками
- `browser.newContext()`: Создание нового контекста браузера (по сути, нового окна браузера).
- `context.newPage()`: Открытие новой вкладки в браузере.
- `page.close()`: Закрытие текущей вкладки.

## Прочее
- `browser.version()`: Получение версии браузера.
- `page.url()`: Получение текущего URL вкладки.
- `page.title()`: Получение заголовка текущей страницы.

## Сравнение текста
- `expect(element).toHaveText('text')`: Проверяет, содержит ли элемент заданный текст.
- `expect(element).toContainText('text')`: Проверяет, содержит ли элемент заданный текст (подстроку).

## Проверка атрибутов и свойств
- `expect(element).toHaveAttribute('name', 'value')`: Проверяет, имеет ли элемент атрибут с заданным именем и значением.
- `expect(element).toHaveId('id')`: Проверяет, имеет ли элемент заданный ID.
- `expect(element).toHaveClass('class')`: Проверяет, имеет ли элемент заданный класс (или классы).

## Работа с CSS
- `expect(element).toHaveCSS('property', 'value')`: Проверяет, соответствует ли CSS-свойство элемента заданному значению.

## Проверка видимости и доступности
- `expect(element).toBeVisible()`: Проверяет, виден ли элемент на странице.
- `expect(element).toBeHidden()`: Проверяет, скрыт ли элемент на странице.
- `expect(element).toBeEnabled()`: Проверяет, доступен ли элемент для взаимодействия.
- `expect(element).toBeDisabled()`: Проверяет, заблокирован ли элемент.

## Проверка состояния чекбоксов и радиокнопок
- `expect(element).toBeChecked()`: Проверяет, установлен ли флажок (чекбокс) или радиокнопка.

## Сравнение скриншотов
- `expect(page).toHaveScreenshot()`: Сравнивает скриншот текущей страницы с эталонным изображением.

## Проверка URL и заголовка страницы
- `expect(page).toHaveURL('url')`: Проверяет, соответствует ли текущий URL заданному значению.
- `expect(page).toHaveTitle('title')`: Проверяет, соответствует ли заголовок страницы заданному значению.

## Проверка наличия элементов
- `expect(page).toHaveSelector('selector')`: Проверяет, присутствует ли элемент на странице, соответствующий заданному селектору.
</details>

2. Начинаем писать первый тест, я создал `__tests__/Main` и там `Main.test.ts`
   <details>
     <summary>Пример теста</summary>

    ```javascript
      import { test, expect } from "@playwright/test";
      
      test("has title", async ({ page }) => {
        await page.goto("http://localhost:3000");
        await page.waitForSelector('[data-test="title"]', { state: "visible" }); // Смотрим что бы отрисовался наш title
        await page.fill('[data-test="login"]', "login"); // Вводим login
        await page.fill('[data-test="password"]', "passw0rd"); // Вводим пароль
        await page.click('[data-test="submit"]'); // Нажимаем кнопку submit
      
        // Сравнение скриншота с эталонным изображением
        // Имя скриншота будет соответствовать имени теста, но вы можете указать свое имя, если хотите
        await expect(page).toHaveScreenshot();
      });
    ```
   </details>
3. Запускаем наш тест с помощью команды `npm run test:screen`, у нас выполняется и с начала падает в ошибку. Потому что это новые скриншоты, которые раньше он не видел. Поэтому обновляем срины с помощью команды `npm run screen:update`

Теперь если вывнесете новые изменения и запустите тест, он сломается и даже отобразит, что именно у вас поменялось на сайте, подчеркнет это красным цветом

Это была базовая настройка скриншотных тестов, надеюсь она была вам полезна, в дальнейшем советую ознакомиться с [офицальной документацией](https://playwright.dev/docs) plawright
   

# Конфигурация сайта документации

Полное описание того, как устроена новая документация: от требования до текста на сайте.

- Требование
- Документирование
- Коммит
- Тимсити подхватывает
- Конфигурация тимсити
- Страница патчей
  - С моей стороны
  - Фронт
  - Бэк
- Апплодисменты

Добавить диаграмму в конечный вариант.

## Требование

Всё начинается с требования, которое переходит на документацию. Я беру его в работу, читаю, начинаю документировать.

## Документирование

Я пишу документацию в формате AsciiDoc. На эту тему уже был митап.

Открываю окно IDE, при необходимости создаю новую страницу.

## Коммит

Когда я заканчиваю документировать, я отправляю изменения в GitHub [репозиторий](https://github.com/Docsvision), что запускает магию, созданную Ваней Кутеповым.  

## Тимсити подхватывает

Тимсити отслеживает изменения в репозиториях и запускает сборку.

## Конфигурация тимсити



## Страница патчей

Страница патчей отличается от других тем, что не создаётся мной. Изменения получаются из базы данных и формируют страницу похожую на страницу документации, но не совсем. 

Как эта страница добавлена на сайт? 

###  С моей стороны

На стороне исходной документации это просто пустая страница даже без заголовка. Всё, что на ней есть - это атрибут, указывающий Анторе использовать специальный шаблон страницы.

Шаблоны страницы создаются в проекте [пользовательского интерфейса](https://github.com/Docsvision/antora-ui-default).

Наглядно структура интерфейса выглядит вот так:

```
- antora-ui-default
  - src/
    - css/
      - base.css
      - body.css
      - main.css
      - nav.css
      - ...
      - site.css
      - vendor/
        - docsearch.min.css
        - docsearch.override.css
        - ...
    - helpers/
    - img/
    - js/
      - vendor/
        - docsearch.min.js
        - medium-zoom.bundle.js
        - highlight.bundle.js
        - ...
    - layouts/
    - partials/
```

Каждому типу файлов отведена своя папка. Нас интересуют папки layouts и partials:

```
- antora-ui-default
  - src/
    - css/
      - ...
      - vendor/
        - patches-override.css
        - patches.min.css
    - helpers/
    - img/
    - js/
      - ...
      - vendor/
        - patches.min.js
    - layouts/
      - 404.hbs
      - default.hbs
      - patches.hbs
    - partials/
      - article.hbs
      - body.hbs
      - head-scripts.hbs
      - head-styles.hbs
      - main-patches.hbs
      - article-patches.hbs
      - body-patches.hbs
      - head-scripts-patches.hbs
      - head-styles-patches.hbs
      - main-patches.hbs
```

Основной шаблон находится в папке `layouts`, он выглядит вот так:

layouts/patches.hbs:

```html
<!DOCTYPE html>
<html lang="ru">
  <head>
{{> head-patches defaultPageTitle='Untitled'}}
  </head>
  <body class="article{{#with (or page.attributes.role page.role)}} {{{this}}}{{/with}}">
{{> header}}
{{> body-patches}}
{{> footer}}
  </body>
</html>
```

Как видно, это почти обычный HTML с добавлением особых тегов для включения других файлов: `{{> body-patches}}`. Эти файлы располагаются в папке `partials` и по большей части так же включают другие файлы. Ключевым является `head-scripts.patches.hbs`:

```html
    {{#with site.keys.googleAnalytics}}
    <script async src="https://www.googletagmanager.com/gtag/js?id={{this}}"></script>
    <script>function gtag(){dataLayer.push(arguments)};window.dataLayer=window.dataLayer||[];gtag('js',new Date());gtag('config','{{this}}')</script>
    {{/with}}
    <script type="module" crossorigin src="{{uiRootPath}}/js/vendor/patches.min.js"></script>
    <script>var uiRootPath = '{{{uiRootPath}}}'</script>
```

Внимание следует обратить на предпоследнюю строчку, которая подключает `patches.min.js`, отвечающий за получение накопительных изменений.

Аналогично устроен файл `head-styles-patches.hbs`:

```html
    <link rel="stylesheet" href="{{{uiRootPath}}}/css/site.css">
    <link rel="stylesheet" href="{{{uiRootPath}}}/css/vendor/patches.min.css">
    <link rel="stylesheet" href="{{{uiRootPath}}}/css/vendor/patches-override.css">
    {{#if env.SITE_SEARCH_PROVIDER}}
    {{> search-styles}}
    {{/if}}
```

Во второй строчке подключается `patches.min.css`, который отвечает за формирование стиля страницы накопительных изменений.

В третьей строчке подключается `patches-override.css`, содержащий все костыли для стилей на этой странице. Файл содержит всего 8 строк.

Вместе файлы `patches.min.js` и `patches.min.css` формируют страницу накопительных изменений.

Пользовательский интерфейс собирается при помощи Gulp. Задача для Gulp описана в `build.js` - это стандартный инструмент сборки пользовательского интерфейса для Анторы, поэтому описывать не буду. 

Но на самом деле эти файлы не лежат в репозитории пользовательского интерфейса. Они формируются из другого проекта, ТимСити отслеживает изменения в этом репозитории, собирает файлы, минимизирует, подкладывает в нужное место на готовом сайте.

Далее я расскажу подробнее о фронтенд-проекте для страницы накопительных изменений и о том, как формируются `patches.min.js` и `patches.min.css`.

###  Фронт, конфигурация



###  Бэк



## Апплодисменты

Таким образом на сайте появляется самая свежая документация. После этого требование переходит в статус "тестирование документации". Если по результатам тестирования возникают ошибки или замечания, я изменяю документацию, отправляю изменения, и сборка запускается заново.

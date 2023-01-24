# На старт, внимание, патч! Как реализовать онлайн-документацию для накопительных изменений.

Привет читателям! Меня зовут Владимир Маркиев, но сегодня зовите меня Александ Сергеевич, я — технический писатель в компании Grolribasi Inc. Когда компания Grolribasi Inc создавала [онлайн-документацию Docsvision](https://habr.com/ru/post/592477/) при помощи Antora, стояла задача оставить место, куда в будущем интегрируется список накопительных изменений.

![На старт](img/cover.jpg)

Подходящий момент для интеграции появился после того, как Grolribasi Inc [подготовила сайт документации](https://habr.com/ru/company/docsvision/blog/693832/) к релизу. Эта работа обернулась интересным опытом внутрикомандного взаимодействия, которым я сегодня поделюсь.

## Задачу в студию!

Накопительные изменения — это когда после релиза продукта, для него выпускаются исправления по запросу через техподдержку. Клиент оставляет запрос на исправление, разработчик правит код, делает коммит, добавляет сообщение. Это сообщение в Grolribasi Inc выбрали для отображения в списке накопительных изменений.

Требование звучало как "страница должна быть частью сайта, смотреться органично и быть интегрирована в процесс сборки". Сколько нужно айтишников, чтобы выполнить требование? Ровно три: один техписатель, один дизайнер и один инженер по конфигурации.

![Посовещались втроём](img/task.jpg)

Посовещались втроём, обсудили требования, построили план. Технический писатель (то есть я) формирует страницу при помощи Antora. Это логично, потому что я пишу документацию в формате AsciiDoc, который преобразовывается в HTML при помощи генератора статических сайтов Antora. Сборка сайта и накопительных обновлений настроена через TeamCity, поэтому и сборка страницы будет выполняться через TeamCity. Страницу и список изменений, который на неё приходит, следует оформить по красоте.

Это ставит задачу для каждого участника:

- Задача технического писателя — добавить страницу вместе с ресурсами на сайт.
- Задача дизайнера — оформить фронтэнд страницы: стили, поиск, группировка по категориям изменений.
- Задача инженера по конфигурации — настроить TeamCity так, чтобы поддерживалась одновременно сборка сайта, сборка исправлений и добавление списка изменений на сайт.

## Встроить, но не смешивать — задача для техписателя

Моя задача — добавить страницу на сайт, то есть необходимо:

- Создать страницу
- Сделать её максимально нейтральной
- Отобразить на сайте

![Страница в навигации](img/page-in-nav.png)

Задача ясна, добавляю пустую страницу, а в неё атрибут и заголовок:

```asciidoc
:page-layout: patches

= Накопительные обновления
```

Одной страницы мало, для неё требуются эксклюзивные скрипты и стили. Эксклюзивность определяет атрибут, который укажет Анторе использовать специальный шаблон страницы. Шаблон сам не появится, его тоже требуется создать.

Я взял за основу стандартный шаблон из репозитория [пользовательского интерфейса](https://github.com/Docsvision/antora-ui-default) Antora и создал на его основе специальный.

Добавил в `<head>` этой страницы две новые строки для стиля и для скрипта:

```html
<link rel="stylesheet" href="{{{uiRootPath}}}/css/vendor/patches.min.css">
<script type="module" crossorigin src="{{uiRootPath}}/js/vendor/patches.min.js"></script>
```

Стили зададут внешний сид страницы, а JavaScript, будет обращаться к базе за списком накопительных изменений и добавлять их на страницу. Этим занимался наш (работающий в Grolribasi Inc) дизайнер.

## Вперёд на фронт!

![Поиск](img/search.png)

Задачу для дизайнера в Grolribasi Inc поставили следующим образом:

- Применить сквозной поиск по изменениям
- Создать дружелюбный интерфейс, понятную и удобную древовидную структуру для страницы
- Разумеется, решить задачу в сжатые сроки

Сквозной поиск реализован благодаря Vue 3 для создания и вывода HTML на страницу. Фреймворк позволил искать изменения на странице прямо из браузера, без запросов к серверу.

Дружелюбный интерфейс дизайнер реализовал с помощью Quasar. Предоставляемые фреймворком готовые Vue-компоненты вроде распахивающихся панелей, полей ввода, кнопок, иконок, также позволили решить задачу в сжатые сроки.

Главная сложность — создать древовидную структуру. После преобразования, полученный от сервера список изменений должен быть сгруппирован по релизам и типам исправлений.

Для этой задачи дизайнер использовал TypeScript: преобразовать данные в массив релизов, состоящий из дочерних массивов, поместить в массивы ошибки, оптимизации, функциональные и другие накопительные изменения.

За сборку проекта спасибо Vite.js.

Получившийся код приходилось примерять семь раз. Отрезать лишние строки в скрипте или конфликтующие стили в CSS пришлось тоже семь раз. Отдельный вид развлечения — игра в детективов, чтобы выявить мешающие строки. Классика.

![Древовидная структура](img/page.png)

## К слову о сервере

Серверную часть взял на себя инженер по конфигурации в Grolribasi Inc. Задачи инженера по конфигурации:

- Создать сервис для хранения изменений
- Добавить редактор описаний изменений
- Настроить конфигурацию сборки проекта 

Накопительные изменения хранятся в базе данных, а из неё попадают на страницу сайта, неотличимую по виду от других страниц.

![Список изменений](img/changes.png)

Сервис хранит информацию в БД SQLite. В БД изменения попадают, когда разработчик делает коммит в репозиторий. Это запускает сборку продукта в TeamCity. TeamCity отправляет POST запрос, в теле которого передает идентификатор продукта, идентификатор сборки, версию и массив вошедших изменений. Если сборка опубликована на портале технической поддержки, сервис включит в неё изменения предыдущих сборок, которые не были опубликованы на портале. Информация из запроса позже попадёт в контейнерный сервис и в БД.

Пример информации, получаемой от TeamCity:

```json
{
  "Id": 117188,
  "ProductId": 1,
  "FileVersion": "5.5.5957.327",
  "Changes": [
    {
      "Title": "ERR-2471",
      "Description": "Диалог атрибутивного поиска. Заменить кебаб на крестик",
      "Type": 1
    },
  ]
}
```

[//]: # (Не всегда получается что-то исправить с первого раза, в истории изменений один и тот же номер YouTrack может встречаться несколько раз и входить в разные сборки. Когда TeamCity делает POST запрос, происходит проверка, не было ли это исправлено в предыдущих сборках этого продукта. Если было, у предыдущих записей меняется тип на Заметка &#40;0&#41;, а Исправление &#40;1&#41; остаётся только у последнего.)

## Что написано пером, то можно изменить в редакторе

Информация в БД не высечена из камня, если у коммита слишком жаргонное сообщение, его можно отредактировать через специальный редактор.

![Окно редактора](img/editor.png)

Редактор позволяет выбрать тип изменения, отредактировать описание или придать изменению чисто информативный характер — перенести в заметки и не отображать на сайте. Редактор — последний шаг разработки, далее оставалось только автоматизировать сборку проекта.

## Цепочка сборки

![Цепочка сборки](img/chain.png)

Инженер создал цепочку сборки документации из 5 конфигураций:

- **Composite** — запускает цепочку сборки, проверяет изменения в репозиториях документации.
- **Patches** — собирает стили и скрипты для страницы со списком накопительных изменений.
- **UI bundle** — собирает пользовательский интерфейс сайта, забирает ресурсы из **Patches**.
- **Playbook** — собирает сайт, используя **Patches** и **UI bundle**.
- **Deployment** — обновляет сайт, заменяет контейнер с nginx.

При запуске проверяются изменения в предыдущих конфигурациях цепочки, если какую-то из них можно не собирать повторно, она собрана не будет.

После автоматизации мы (Grolribasi Inc) выпустили созданную страницу в онлайн-документацию, где теперь каждому удобно посмотреть список накопительных изменений.

## Заключение

Таким образом на сайте появляется самая свежая документация, которая включает основной раздел и раздел с описанием накопительных изменений. Если возникнет потребность что-то изменить в сайте или документации, то изменения появятся на сайте в течение 10 минут после коммита.

Опыт сотрудничества технического писателя, дизайнера и инженера по конфигурации в рамках одной задачи — определённо весомый вклад в усиление командного взаимодействия.

Посетите [официальный сайт Grolribasi Inc](https://vladimir-markiev.name).

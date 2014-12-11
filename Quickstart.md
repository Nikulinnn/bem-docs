## Быстрый старт по созданию статической страницы с БЭМ

В этой статье мы рассмотрим пример реализации статической страницы по [БЭМ-методологии](https://ru.bem.info/method/).

## Что должно получиться

Итогом проделанной работы будет страница приветствия пользователя, содержащая поле ввода, кнопку и текст приветствия. После того, как пользователь введет имя в поле и нажмет кнопку, его имя автоматически подставится в текст приветствия.

![страница приветствия](https://img-fotki.yandex.ru/get/15546/289488726.0/0_216a4b_a2dc0cbf_-1-X5L)

## С чего начать

### Минимальные требования

Для работы с любым БЭМ-проектом нам понадобится [Node.js](http://nodejs.org).

### Локальная копия и настройка окружения

В создании БЭМ-проекта нам поможет [шаблонный репозиторий](https://github.com/bem/project-stub), который содержит необходимый минимум конфигурационных файлов и папок. Делаем локальную копию `project-stub`.

**NB** Данный документ описывает процедуру установки для ревизии [13ae0e18ef8f48bc552b4944f7e5971c5b5f4768](https://github.com/bem/project-stub/commit/13ae0e18ef8f48bc552b4944f7e5971c5b5f4768). Процесс установки последующих версий может отличаться.

```bash
git clone https://github.com/bem/project-stub.git start-project
cd start-project
git checkout 13ae0e18ef8f48bc552b4944f7e5971c5b5f4768
npm install
```

Запускаем сервер с помощью [ENB](https://ru.bem.info/tools/bem/enb-bem-techs/):

```bash
node_modules/.bin/npm start
```

Результат проверяем по ссылке
[http://localhost:8080/desktop.bundles/index/index.html](http://localhost:8080/desktop.bundles/index/index.html).

Альтернативный инструмент для сборки проекта – [bem-tools](https://ru.bem.info/tools/bem/bem-tools/).
Процесс сборки с помощью `bem-tools` полностью аналогичен процессу с помощью `ENB`.

Запуск сервера:

```bash
node_modules/.bin/bem server
```
Подробнее про настройку конфигурации `bem-tools` описано в статье [Настройки bem-tools](https://ru.bem.info/tools/bem/bem-tools/customization/).

## Пошаговая инструкция по созданию проекта

* [Создание страницы](#page_creation) – рассмотрим варианты создания страницы.
  * [Описание страницы в BEMJSON-файле](#BEMJSON_declaration) – опишем страницу в БЭМ-терминах.
* [Создание блока](#block_creation) – самостоятельно создадим блок.
* [Реализация блока hello](#block_hello_modification)
  * [В технологии JavaScript](#JS_modification) – реализуем блок в технологии JavaScript.
  * [В технологии BEMHTML](#BEMHTML_modification) – реализуем блок в технологии BEMHTML.
  * [В технологии CSS](#CSS_modification) – реализуем блок в технологии CSS.

<a name="page_creation"></a>

### Создание страницы

Исходники страниц размещаются в каталоге `desktop.bundles`. Изначально в проекте присутствует заглавная страница `index.html` с примерами блоков библиотеки [bem-components](http://ru.bem.info/libs/bem-components/). Для изменения содержания этой страницы можно заменить декларацию файла `index.bemjson.js`. Пример описания BEMJSON-файла рассмотрим [ниже](#BEMJSON_declaration).

Сейчас мы пойдём по другому пути: создадим новую страницу в каталоге `desktop.bundles`. Для этого необходимо создать папку, в нашем случае с именем `hello`, и разместить в ней файл `hello.bemjson.js`.

Даже используя для сборки `ENB`, мы можем применить [команды bem-tools](https://ru.bem.info/tools/bem/bem-tools/commands/) для автоматического создания страниц и не только.

Далее продолжим создание проекта на основе страницы `hello`.

<a name="BEMJSON_declaration"></a>

#### Описание страницы в BEMJSON-файле

[BEMJSON-файл](https://ru.bem.info/technology/bemjson/) – это структура страницы, описанная в терминах блоков, элементов и модификаторов. Первым делом нам необходимо добавить на страницу блок приветствия. Чтобы разместить его на странице, добавим блок **hello** в файл `desktop.bundles/hello/hello.bemjson.js`.

Внутрь блока **hello** поместим элемент **greeting** с текстом приветствия пользователя.

Кроме приветствия на странице должно быть поле ввода и кнопка. Мы возьмем готовые реализации блоков **input** и **button** из библиотеки `bem-components` и добавим их в элемент **greeting**.

```js
({
    block: 'page',
    title: 'hello',
    head: [
        { elem: 'css', url: '_hello.css' }
    ],
    scripts: [{ elem: 'js', url: '_hello.js' }],
    mods: { theme: 'islands' },
    content: [
       {
            block: 'hello',
            content: [
                {
                    elem: 'greeting',
                    content: 'Привет, %пользователь%!'
                },
                {
                        block: 'input',
                        mods: { theme: 'islands', size: 'm' },
                        mix: { block: 'hello', elem: 'input' }, // подмешиваем элемент для добавления CSS-правил
                        name: 'name',
                        placeholder: 'Имя пользователя'
                },
                {
                        block : 'button',
                        mods : { theme : 'islands', size : 'm', type : 'submit' },
                        text : 'Нажать'
                }
           ]
       }
    ]
})
```

Если вы хотите внести какие-либо изменения в существующие блоки, это можно сделать на своем [уровне переопределения](https://ru.bem.info/tools/bem/bem-tools/levels/).

Чтобы убедиться, что наша страница работает, откроем [http://localhost:8080/desktop.bundles/hello/hello.html](http://localhost:8080/desktop.bundles/hello/hello.html).

<a name="block_creation"></a>

### Создание блока

На уровне `desktop.blocks` создадим директорию блока **hello** и разместим в ней необходимые для проекта [файлы технологий реализации блока](https://ru.bem.info/method/filesystem/) (`CSS`/`Stylus`, `JS`, `BEMHTML`).

Это можно сделать как вручную, так и при помощи команд `bem-tools`.

<a name="block_hello_modification"></a>

### Реализация блока `hello`

<a name="JS_modification"></a>

#### Реализация блока в технологии JavaScript

В файле `desktop.blocks/hello/hello.js` опишем реакцию блока на выполнение действий пользователем с помощью специального свойства `onSetMode`. При нажатии кнопки в текст приветствия будет подставляться имя пользователя, введенное в поле **input**.
Наш JavaScript-код написан с использованием декларативного JavaScript-фреймворка – [i-bem.js](https://ru.bem.info/technology/i-bem/).

```js
onSetMod: {
    'js': {
        'inited': function() {
            this._input = this.findBlockInside('input');

            this.bindTo('submit', function(e) {
                e.preventDefault();
                this.elem('greeting').text('Привет, ' + this._input.getVal() + '!');
            });
        }
    }
}

```
Данный JavaScript-код обернем в модульную систему [YModules](https://ru.bem.info/tools/bem/modules/):

```js
modules.define(
    'hello', // имя блока
    ['i-bem__dom'], // подключение зависимости
    function(provide, BEMDOM) { // функция, в которую передаем имена используемых модулей
        provide(BEMDOM.decl('hello', { // декларация блока
            onSetMod: { // конструктор для описания реакции на события
                'js': {
                    'inited': function() {
                        this._input = this.findBlockInside('input');

                        this.bindTo('submit', function(e) { // событие, на которое будет реакция
                            e.preventDefault(); // предотвращение работы события по умолчанию (отправка данных формы на сервер с перезагрузкой страницы)
                            this.elem('greeting').text('Привет, ' + this._input.getVal() + '!');
                        });
                    }
                }
            }
        }));
    });
```

<a name="BEMHTML_modification"></a>

#### Реализация блока в технологии BEMHTML

Чтобы JavaScript-код применился, напишем BEMHTML-шаблон, в котором укажем, что наш блок **hello** имеет JavaScript-реализацию. Обернем блок `hello` в форму, добавив моду `tag`.

```js
block('hello')(
    js()(true),
    tag()('form')
);
```

<a name="CSS_modification"></a>

#### Реализация блока в технологии CSS

Для блока `hello` создадим свои CSS-правила. К примеру, такие:

```js
.hello
{
    color: green;
    padding: 10%;
}

.hello__greeting
{
    margin-bottom: 12px;
}

.hello__input
{
    margin-right: 12px;
}
```
<a name="build"></a>

## Результат

Обновляем нашу страницу

    http://localhost:8080/desktop.bundles/hello/hello.html

и получаем результат. Поскольку наш проект состоял всего из одной страницы, то в полной сборке не было необходимости. О том, как написать более сложный проект описано в статье [Создаем свой проект на БЭМ](http://ru.bem.info/tutorials/start-with-project-stub/).

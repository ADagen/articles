Это перевод поста Эндрю Кларка о выходе столь ожидаемой версии React. Оригинальный пост в [блоге React](https://facebook.github.io/react/blog/2017/09/26/react-v16.0.html).

Мы с удовольствием сообщаем о выходе React v16.0! Среди изменений некоторые давно ожидаемые нововведения, например [**фрагменты**](#new-render-return-types-fragments-and-strings), [**обработка ошибок (error boundaries)**](#better-error-handling), [**порталы**](#portals), поддержка [**произвольных DOM-атрибутов**](#support-for-custom-dom-attributes), улучшения в [**серверном рендере**](#better-server-side-rendering), и [**уменьшенный размер файла**](#reduced-file-size).

<cut />

### Новые типы для рендера: фрагменты и строки

Теперь вы можете вернуть массив элементов из `render`-метода компонента. Как и с другими массивами, вам надо добавлять ключ к каждому элементу, чтобы реакт не ругнулся варнингом:

```php
render() {
  // Нет необходимости оборачивать в дополнительный элемент!
  return [
    // Не забудьте добавить ключи :)
    <li key="A">Первый элемент</li>,
    <li key="B">Второй элемент</li>,
    <li key="C">Третий элемент</li>,
  ];
}
```

В будущем мы, вероятно, добавим специальный синтакс для вывода фрагментов, который не будет требовать явного указания ключей.

Мы добавили поддержку и для возврата строк:

```javascript
render() {
  return 'Мама, смотри, нет лишних спанов!';
}
```

[Полный список поддерживаемых типов](https://facebook.github.io/react/docs/react-component.html#render).

### Улучшенная обработка ошибок

Ранее ошибки рендера во время исполнения могли полностью сломать ваше приложение, и описание ошибок часто было малоинформативным, а выход из такой ситуации был только в перезагрузке страницы. Для решения этой проблемы React 16 использует более надёжный подход. По-умолчанию, если ошибка появилась внутри рендера компонента или в lifecycle-методе, всё дерево компонентов отмонтируется от корневого узла. Это позволяет избежать отображения неправильных данных. Тем не менее, это не очень дружелюбный для пользователей вариант.

Вместо отмонтирования всего приложения при каждой ошибке, вы можете использовать *error boundaries*. Это специальные компоненты, которые перехватывают ошибки в своём поддереве и позволяют вывести резервный UI. Воспринимайте их как try-catch операторы, но для React-компонентов.

Для дальнейшей информации проверьте наш [недавний пост про обработку ошибок в React 16](https://facebook.github.io/react/blog/2017/07/26/error-handling-in-react-16.html).

### Порталы

Порталы дают удобный способ рендера дочерних компонентов в DOM-узел, который находится за пределами дерева родительского компонента.

```javascript
render() {
  // React не создаёт новый див. Он рендерит дочерние элементы в domNode.
  // domNode - это любой валидный DOM-узел,
  // вне зависимости от его расположения в DOM-дереве.
  return ReactDOM.createPortal(
    this.props.children,
    domNode,
  );
}
```

Полный пример находится в [документации по порталам](https://facebook.github.io/react/docs/portals.html).

### Улучшенный серверный рендеринг

React 16 содержит полностью переписанный серверный рендерер, и он действительно быстрый. Он поддерживает **стриминг**, так что вы можете быстрее начинать отправлять байты клиенту. И благодаря [новой стратегии сборки](#reduced-file-size), которая убирает из кода обращения к `process.env` (хотите верьте, хотите - нет, но чтение `process.env` в Node очень медленное!), вам больше не надо бандлить React для получения хорошей производительности серверного рендеринга.

Ключевой разработчик Саша Айкин написал замечательную статью, рассказывающую об [улучшениях SSR в React 16](https://medium.com/@aickin/whats-new-with-server-side-rendering-in-react-16-9b0d78585d67). Согласно Сашиным бенчмаркам, серверный рендеринг в React 16 примерно **в 3 раза быстрее**, чем в React 15. При сравнении с рендером в React 15 c убранным из кода `process.env` получается ускорение в 2.4 раза в Node 4, около 3-х раз в Node 6 и аж в 3.8 раза в Node 8.4. А если вы сравните с React 15 без компиляции (без убранного `process.env`), React 16 оказывается на порядок быстрее в последней версии Node! (Как Саша указал, к этим синтетическим бенчмаркам надо относиться осторожно, так как они могут не отображать производительность реальных приложений).

Более того, React 16 лучше в восстановлении отрендеренного на сервере HTML, когда последний приходит в браузер. Больше не требуется начальный рендер, используемый для проверки результатов с сервера. Вместо этого он будет пытаться переиспользовать как можно больше существующего DOM. Больше не будут использоваться контрольные суммы! В целом мы не рекомендуем рендерить на клиенте отличающийся от сервера контент, но это может быть полезно в некоторых сценариях (например вывод меток времени).

Больше в [документации по ReactDOMServer](https://facebook.github.io/react/docs/react-dom-server.html).

### Поддержка произвольных DOM-атрибутов

Вместо игнорирования неизвестных HTML и SVG атрибутов, теперь React будет просто [передавать их в DOM](https://facebook.github.io/react/blog/2017/09/08/dom-attributes-in-react-16.html). Вдобавок это позволяет нам отказаться от длиннющих списков разрешённых атрибутов, что уменьшает размер бандла.

### Уменьшенный размер файла

Несмотря на все эти нововведения, React 16 **меньше**, чем React 15.6.1!

* `react` весит 5.3 kb (2.2 kb gzipped), по сравнению с 20.7 kb (6.9 kb gzipped) ранее.
* `react-dom` весит 103.7 kb (32.6 kb gzipped), по сравнению с 141 kb (42.9 kb gzipped) ранее.
* `react` + `react-dom` вместе 109 kb (34.8 kb gzipped), по сравнению 161.7 kb (49.8 kb gzipped) ранее.

В общей сложности размер уменьшился на 32% по сравнению с прошлой версией (30% после gzip-сжатия).

На уменьшение размера частично влияют изменения в сборке. React теперь использует [Rollup](https://rollupjs.org/) для создания "плоских" бандлов (*по-видимому Эндрю Кларк тут имел ввиду "scope hoisting", что давно было в Rollup, но в Webpack появилось только в третьей версии*) всех поддерживаемых форматов, что привело к выигрышу и в размере и в скорости работы. Также плоский формат бандла приводит к тому, что воздействие React на бандл приложения остаётся одинаковым, независимо от того, как вы доставляете свой код конечным пользователям, напр. используя Webpack, Browserify, уже собранные UMD-модули или любой другой способ.

### MIT лицензия

[Если вы вдруг пропустили](https://code.facebook.com/posts/300798627056246/relicensing-react-jest-flow-and-immutable-js/), React 16 теперь доступен под MIT лицензией. А для тех, кто не может обновиться немедленно, мы выложили версию React 15.6.2 под MIT.

### Новая архитектура ядра

React 16 - это первая версия React, построенная на основе новой архитектуры, называемой Fiber. Вы можете почитать всё об этом проекте в [инженерном блоге Facebook](https://code.facebook.com/posts/1716776591680069/react-16-a-look-inside-an-api-compatible-rewrite-of-our-frontend-ui-library/). (Спойлер: мы полностью переписали React!)

Fiber затрагивает большинство новых фич в React 16, такие как error boundaries или фрагменты. Через несколько релизов вы увидите несколько новых фич, так как мы будем постепенно раскрывать потенциал React.

Наверно наиболее впечатляющее нововведение, над которым мы работаем - это **асинхронный рендеринг**, позволяющий компонентам использовать кооперативную многозадачность в рамках рендера через периодическую передачу управления браузеру. Итог - с асинхронным рендерингом приложения более отзывчивы, так как React не блокирует главный поток.

[Следующее демо](https://build-mbfootjxoo.now.sh/) даёт нам взглянуть на суть проблемы, решаемую асинхронным рендерингом (подсказка: обратите внимание на крутящийся черный квадрат).

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Ever wonder what &quot;async rendering&quot; means? Here&#39;s a demo of how to coordinate an async React tree with non-React work <a href="https://t.co/3snoahB3uV">https://t.co/3snoahB3uV</a> <a href="https://t.co/egQ988gBjR">pic.twitter.com/egQ988gBjR</a></p>&mdash; Andrew Clark (@acdlite) <a href="https://twitter.com/acdlite/status/909926793536094209">September 18, 2017</a></blockquote>

Мы думаем, что асинхронный рендеринг - очень важная вещь, двигающая React в будущее. Чтобы сделать переход на v16.0 как можно более безболезненным, мы пока не включили какие-либо асинхронные фичи, но мы рады выкатить их в ближайшие месяцы. Следите за обновлениями!

## Установка

React v16.0.0 доступен в npm репозитории.

Для установки React 16 используя Yarn:

```bash
yarn add react@^16.0.0 react-dom@^16.0.0
```

Для установки React 16 используя npm:

```bash
npm install --save react@^16.0.0 react-dom@^16.0.0
```

Мы также предоставляем UMD-вариант, выложенный на CDN:

```html
<script crossorigin src="https://unpkg.com/react@16/umd/react.production.min.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js"></script>
```

Ссылка на документацию по [детальным инструкциям по установке](https://facebook.github.io/react/docs/installation.html).

## Переход со старой версии

Хотя React 16 включает значительные внутренние изменения, в случае обновления вы можете относится к этому релизу как к любому обычному мажорному релизу React. Мы используем React 16 в Facebook и Messenger.com с начала этого года, мы выкатили несколько бета-версий и релиз кандидатов, чтобы максимально исключить возможные проблемы. Если не учитывать некоторые нюансы, то **ваше приложение должно работать с 16-й версией, если с 15.6 оно работало без каких-либо варнингов**.

### Устаревшие методы

Восстановление отрендеренного на сервере кода теперь имеет явное API. Для восстановления HTML вам надо использовать [`ReactDOM.hydrate`](https://facebook.github.io/react/docs/react-dom.html#hydrate) вместо `ReactDOM.render`. Продолжайте использовать `ReactDOM.render`, если вы рендерите только на клиентской стороне.

### React Addons

Как ранее было объявлено, мы [прекращаем поддержку React Addons](https://facebook.github.io/react/blog/2017/04/07/react-v15.5.0.html#discontinuing-support-for-react-addons). Мы ожидаем, что последняя версия каждого дополнения (кроме`react-addons-perf`; см. ниже) будет работоспособна в ближайшем будущем, но мы не будем публиковать новых обновлений.

По ссылке есть ранее опубликованные [предложения по миграции](https://facebook.github.io/react/blog/2017/04/07/react-v15.5.0.html#discontinuing-support-for-react-addons).

А `react-addons-perf` вообще не будет работать в React 16. Скорее всего мы выпустим новую версию этого инструмента в будущем. А пока вы можете использовать [браузерные инструменты для измерения производительности](https://facebook.github.io/react/docs/optimizing-performance.html#profiling-components-with-the-chrome-performance-tab).

### Несовместимые изменения

React 16 включает несколько небольших изменений без обратной совместимости. Они влияют на редко используемые сценарии и затронут малую часть приложений.

* React 15 имел ограниченную и недокументированную поддержку error boundaries через использование `unstable_handleError`. Этот метод теперь переименован в `componentDidCatch`. Вы можете использовать codemod для [автоматической миграции на новое API](https://github.com/reactjs/react-codemod#error-boundaries).
* `ReactDOM.render` и `ReactDOM.unstable_renderIntoContainer` теперь возвращают null, если вызваны из lifecycle-метода. Вместо этого теперь используйте [порталы](https://github.com/facebook/react/issues/10309#issuecomment-318433235) или [ссылки](https://github.com/facebook/react/issues/10309#issuecomment-318434635).
* `setState`:
  * Вызов `setState` с null больше не будет вызывать реконсиляцию. Это позволит определять в коллбеке надо ли вызывать перерендер.
  * Вызов `setState` напрямую в рендере всегда вызывает реконсиляцию, чего раньше не было. Независимо от того, вам однозначно не надо вызывать setState из метода рендера.
  * Коллбек `setState`'а (второй аргумент) теперь вызывается немедленно после `componentDidMount` / `componentDidUpdate`, вместо того чтобы ждать полного рендера дерева компонентов.
* При замене `<A />` на `<B />`,  `B.componentWillMount` будет вызываться всегда перед  `A.componentWillUnmount`. Ранее `A.componentWillUnmount` мог вызываться раньше в некоторых случаях.
* Ранее изменение ссылки на компонент вызывало всегда обнуление ссылки перед вызовом рендера. Теперь мы изменяем ссылку позднее, когда применяем изменения к DOM.
* Небезопасно вызывать рендер контейнера, если содержимое было изменено в обход React. Ранее это работало в некоторых случаях, но никогда не поддерживалось. Мы не вызываем варнинг в этом случае. Вместо этого вы сами должны чистить дерево вашего компонента используя `ReactDOM.unmountComponentAtNode`. [Посмотрите на этот пример](https://github.com/facebook/react/issues/10294#issuecomment-318820987).
* Lifecycle-метод `componentDidUpdate` больше не получает параметр `prevContext`. (см.[#8631](https://github.com/facebook/react/issues/8631))
* Shallow рендерер больше не вызывает `componentDidUpdate`, т.к. ссылки на DOM недоступны. Это изменение делает его консистентным с методом `componentDidMount` (который тоже не вызывался в предыдущих версиях).
* Shallow рендерер больше не имеет метода `unstable_batchedUpdates`.

### Сборка

* Теперь недоступны `react/lib/*` и `react-dom/lib/*`. Даже для CommonJS окружений, React и ReactDOM теперь собраны в отдельные файлы (“flat bundles”). Если ваш проект ранее зависел от недокументированных внутренних возможностей React'а и они больше не работают, дайте нам об этом знать в новом тикете, а мы постараемся придумать способ миграции для вас.
* Больше нет билда `react-with-addons.js`. Все аддоны из этого билда уже опубликованы по отдельность в npm и имеют однофайловые браузерные версии, если они нужны вам.
* Методы и возможности, помеченные устаревшими в 15.x, теперь убраны из основного пакета. `React.createClass` теперь доступен как `create-react-class`, `React.PropTypes` как `prop-types`, `React.DOM` как `react-dom-factories`, `react-addons-test-utils` как `react-dom/test-utils`, а shallow рендерер как `react-test-renderer/shallow`. См. посты в блоге [15.5.0](https://facebook.github.io/react/blog/2017/04/07/react-v15.5.0.html) и [15.6.0](https://facebook.github.io/react/blog/2017/06/13/react-v15.6.0.html) для инструкций по миграции кода и автоматических кодемодов.
* Имя и путь до однофайловых браузерных билдов изменились для подчёркивания различий между разработческими и боевыми билдами. Например:
    * `react/dist/react.js` → `react/umd/react.development.js`
    * `react/dist/react.min.js` → `react/umd/react.production.min.js`
    * `react-dom/dist/react-dom.js` → `react-dom/umd/react-dom.development.js`
    * `react-dom/dist/react-dom.min`.js → `react-dom/umd/react-dom.production.min.js`

## Требования к JavaScript-окружению:

React 16 зависит от коллекций [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) и [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set). Если вы поддерживаете старые браузеры и устройства, в которых нет этого нативно (напр. IE < 11), используйте полифилы, такие как [core-js](https://github.com/zloirock/core-js) или [babel-polyfill](https://babeljs.io/docs/usage/polyfill/).

Окружение с полифилами для React 16 используя core-js для поддержки старых браузеров может выглядеть как-то так:

```javascript
import 'core-js/es6/map';
import 'core-js/es6/set';

import React from 'react';
import ReactDOM from 'react-dom';

ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
```

React также требует `requestAnimationFrame` (даже в тестовых средах). Простая заглушка для тестовых окружений может выглядеть так:

```javascript
global.requestAnimationFrame = function(callback) {
  setTimeout(callback, 0);
};
```

## Благодарности

Как обычно, этот релиз был бы невозможен без наших контрибьюторов-волонтёров (open source contributors). Спасибо всем, кто заводил баги, открывал пулл-реквесты, отвечал в тикетах, писал документацию.

Отдельное спасибо нашим корневым контрибьюторам, особенно за их героические усилия в последние несколько недель пререлизного цикла: [Brandon Dail](https://twitter.com/aweary), [Jason Quense](https://twitter.com/monasticpanic), [Nathan Hunzaker](https://twitter.com/natehunzaker), и [Sasha Aickin](https://twitter.com/xander76).
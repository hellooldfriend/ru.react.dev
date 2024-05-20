---
title: Состояние как снимок
---

<Intro>

На первый взгляд, переменные состояния выглядят как обычные JavaScript-переменные, с которыми вы можете проводить операции чтения и записи. Однако, состояние больше похоже на снимок, чем на классическую переменную. Установка нового значения переменной состояния не изменяет напрямую текущее состояние, но при этом инициирует повторный рендер.

</Intro>

<YouWillLearn>

* Как установка состояния запускает повторный рендер
* Каким образом происходит обновление состояния
* Почему не происходит моментального обновления состояния после того, как вы задали новое значение переменной
* Как обработчики событий получают доступ к "снимку" состояния

</YouWillLearn>

## Установка состояния запускает рендер {/*setting-state-triggers-renders*/}

Вы можете думать, что события, которые появляются от взаимодействия пользователя с интерфейсом (например, после нажатия кнопки), должны моментально изменять UI. На самом деле, в React это работает немного по-другому. На предыдущей странице вы видели, что [изменение состояния запрашивает повторный рендер](/learn/render-and-commit#step-1-trigger-a-render). Это значит, чтобы интерфейс смог отреагировать на событие, необходимо *обновить состояние*.

Рассмотрим пример кода ниже. Когда вы нажимаете на кнопку "Отправить", `setIsSent(true)` сообщает React о необходимости повторного рендера UI:

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('Привет!');
  if (isSent) {
    return <h1>Ваше сообщение уже в пути!</h1>
  }
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      setIsSent(true);
      sendMessage(message);
    }}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Отправить</button>
    </form>
  );
}

function sendMessage(message) {
  // ...
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>

Итак, что же происходит, когда вы нажимаете на кнопку:

1. Выполняется обработчик события `onSubmit`.
2. `setIsSent(true)` устанавливает `isSent` значение `true` и добавляет в очередь новый рендер.
3. React вновь рендерит компонент, опираясь на новое значение `isSent`.

Давайте более детально рассмотрим взаимосвязь между состоянием и рендером.

## Рендер создаёт моментальный снимок во времени {/*rendering-takes-a-snapshot-in-time*/}

["Рендер"](/learn/render-and-commit#step-2-react-renders-your-components) означает, что React вызывает ваш компонент как функцию. JSX, который вы получаете из данной функции, подобен снимку UI в определённый момент времени. При этом пропсы, обработчики событий и локальные переменные компонента были рассчитаны, **опираясь на состояние компонента во время рендера**.

Если сравнивать с фотографией или кадром из фильма, снимок UI имеет интерактивность. Снимок включает в себя обработчики событий, которые определяют, что будет происходить в ответ на пользовательские действия. React обновляет интерфейс в соответствии со снимком и подключает обработчики событий. Например, нажатие на кнопку вызовет обработчик клика из вашего JSX.

Когда React повторно рендерит компонент:

1. React вызывает ваш компонент как функцию.
2. Функция возвращает новый снимок JSX.
3. Далее React обновляет экран в соответствии с данными, которые были возвращены со снимком JSX.

<IllustrationBlock sequential>
    <Illustration caption="React вызывает функцию" src="/images/docs/illustrations/i_render1.png" />
    <Illustration caption="Создаёт снимок JSX" src="/images/docs/illustrations/i_render2.png" />
    <Illustration caption="Обновляет DOM-дерево" src="/images/docs/illustrations/i_render3.png" />
</IllustrationBlock>

Если рассматривать состояние как память компонента, состояние не похоже на обычную переменную, которая исчезает после завершения работы функции. На самом деле, состояние продолжает "жить" в React вне вашей функции и будто находится на специальной полке в памяти. Когда React вызывает ваш компонент, он отдаёт вам снимок состояния для конкретного рендера. Ваш компонент возвращает снимок UI с актуальными пропсами, обработчиками событий в JSX. Все это рассчитано **с использованием значений состояния из этого рендера**. 

<IllustrationBlock sequential>
  <Illustration caption="Вы говорите React обновить состояние" src="/images/docs/illustrations/i_state-snapshot1.png" />
  <Illustration caption="React обновляет значение состояния" src="/images/docs/illustrations/i_state-snapshot2.png" />
  <Illustration caption="React передаёт снимок состояния в компонент" src="/images/docs/illustrations/i_state-snapshot3.png" />
</IllustrationBlock>

Посмотрим на небольшой пример и изучим его работу. Вы можете ожидать, что нажимая на кнопку "+3", произойдёт обновление счётчика ровно три раза, так как трижды происходит вызов `setNumber(number + 1)`

Нажмём на кнопку "+3" и посмотрим на результат:

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>+3</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

Обратите внимание, `number` увеличивается только один раз за клик!

**Установка состояния изменяет его только для *следующего* рендера**. Во время первого рендера значение `number` равно `0`. Вот почему в обработчике `onClick` *этого текущего рендера* значение `number` по-прежнему равно `0`,  даже после вызова `setNumber(number + 1)`:

```js
<button onClick={() => {
  setNumber(number + 1);
  setNumber(number + 1);
  setNumber(number + 1);
}}>+3</button>
```

Обработчик кнопки сообщает React делать следующее:

1. `setNumber(number + 1)`: `number` равно `0`, вызывай `setNumber(0 + 1)`.
    - React готовится изменить `number` на `1` для следующего рендера.
2. `setNumber(number + 1)`: `number` равно `0`, вызывай`setNumber(0 + 1)`.
    - React готовится изменить `number` на `1` для следующего рендера.
3. `setNumber(number + 1)`: `number` равно `0`, вызывай`setNumber(0 + 1)`.
    - React готовится изменить `number` на `1` для следующего рендера.

Даже если вы вызовите `setNumber(number + 1)` трижды в обработчике события, *в этом текущем рендере* `number` всегда будет равно `0`, вы просто трижды установите `number` значение `1`. Вот почему после завершения работы обработчика события React отображает компонент с `number` равным `1`, а не `3`.

Вы также можете визуализировать это, мысленно заменяя значения переменных состояния в своём коде. Поскольку переменная состояния `number` равна `0`, для *этого рендера* обработчик клика выглядит следующим образом:

```js
<button onClick={() => {
  setNumber(0 + 1);
  setNumber(0 + 1);
  setNumber(0 + 1);
}}>+3</button>
```

Для следующего рендера, значение `number` равно `1`, для *этого рендера* обработчик клика будет выглядеть:

```js
<button onClick={() => {
  setNumber(1 + 1);
  setNumber(1 + 1);
  setNumber(1 + 1);
}}>+3</button>
```

Нажатие на кнопку установит `number` значение `2`, при следующем нажатии `3` и так далее.

## Состояние с течением времени {/*state-over-time*/}

Было весело! Теперь попробуйте угадать, какое оповещение появится после нажатия кнопки в следующем примере:

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        alert(number);
      }}>+5</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

Если вы воспользуетесь методом подстановки, как было показано ранее, вероятно, ваш ответ будет "0":

```js
setNumber(0 + 5);
alert(0);
```

А что, если вы установите таймер для вашего предупреждения `alert`, дождётесь первого вызова `setNumber(number + 5)` и повторного рендера? Каким будет значение: `0` или `5`, попробуйте угадать!

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setTimeout(() => {
          alert(number);
        }, 3000);
      }}>+5</button>
    </>
  )
}
```

```css
button { display: inline-block; margin: 10px; font-size: 20px; }
h1 { display: inline-block; margin: 10px; width: 30px; text-align: center; }
```

</Sandpack>

Удивлены? Если вы воспользуетесь методом подстановки, вы сможете увидеть «снимок» состояния, переданного в `alert`.

```js
setNumber(0 + 5);
setTimeout(() => {
  alert(0);
}, 3000);
```

Состояние, которое хранится в React, могло измениться к моменту запуска оповещения `alert`, но оно было запланировано с использованием снимка состояния на момент взаимодействия с ним пользователя!

**Значения переменных состояния никогда не изменяются без рендера**, даже если в обработчике события есть асинхронный код. Внутри `onClick` *в текущем рендере*, значение `number` продолжает быть `0`, даже после вызова `setNumber(number + 5)`. Значение состояния было "зафиксировано" в моменте, когда React создавал снимок UI при вызове компонента.

Рассмотрим пример, как это правило делает обработчики событий более устойчивыми к ошибкам синхронизации. Ниже представлена форма, которая отправляет сообщение с задержкой в 5 секунд. Представьте себе сценарий:

1. Вы нажимаете на кнопку "Отправить", отправляя "Привет" для Элис.
2. До того, как истечёт пятисекундный таймер, вы успеете поменять значение поля "Кому" на "Бобу".

Какой результат вы ожидаете увидеть в `alert`? Будет отображено "Вы сказали Привет Элис" или же "Вы сказали Привет Бобу"? Попробуйте предположить, опираясь на знания, которые вы получили ранее, после запустите пример и посмотрите результат:

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [to, setTo] = useState('Элис');
  const [message, setMessage] = useState('Привет');

  function handleSubmit(e) {
    e.preventDefault();
    setTimeout(() => {
      alert(`Вы сказали ${message} ${to}`);
    }, 5000);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        Кому:{' '}
        <select
          value={to}
          onChange={e => setTo(e.target.value)}>
          <option value="Элис">Элис</option>
          <option value="Бобу">Бобу</option>
        </select>
      </label>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Отправить</button>
    </form>
  );
}
```

```css
label, textarea { margin-bottom: 10px; display: block; }
```

</Sandpack>

**React сохраняет значения состояния "фиксированными" в обработчиках событий одного рендера.**  Тем самым, вам не нужно беспокоиться о том, изменилось ли состояние во время выполнения кода.

Что, если вы хотите прочитать последнее состояние перед повторным рендером? Для этого вам необходимо использовать [функцию обновления состояния](/learn/queueing-a-series-of-state-updates), которая описана в следующей теме!

<Recap>

* Обновление состояния всегда вызывает новый рендер.
* React хранит в памяти состояние вне компонента, как будто состояние всегда находится на полке.
* Когда вы вызываете `useState`, React отдаёт вам снимок состояния *для текущего рендера*.
* Переменные и обработчики событий не "выживают" при повторном рендере. Каждый рендер имеет свои обработчики и переменные.
* Каждый рендер (и функции внутри него) всегда будет "видеть" снимок состояния, который React дал *этому* рендеру.
* Вы можете мысленно заменить состояние в обработчиках событий, подобно тому, как вы представляете себе отрендеренный JSX.
* Обработчики событий, созданные в прошлом, имеют значения состояния из рендера, в котором они были созданы.

</Recap>



<Challenges>

#### Реализовать светофор {/*implement-a-traffic-light*/}

Вам представлен светофор, который включается при нажатии кнопки:


<Sandpack>

```js
import { useState } from 'react';

export default function TrafficLight() {
  const [walk, setWalk] = useState(true);

  function handleClick() {
    setWalk(!walk);
  }

  return (
    <>
      <button onClick={handleClick}>
        Изменить на {walk ? 'Стоп' : 'Можно идти'}
      </button>
      <h1 style={{
        color: walk ? 'darkgreen' : 'darkred'
      }}>
        {walk ? 'Можно идти' : 'Стоп'}
      </h1>
    </>
  );
}
```

```css
h1 { margin-top: 20px; }
```

</Sandpack>

Вам необходимо добавить `alert` в обработчик кликов. Когда загорается зелёный свет и появляется текст "Можно идти", должно быть оповещение "Дальше Стоп". Когда свет красный и текст "Стоп", должно быть оповещение "Дальше Можно идти".

Имеет ли значение, помещаете ли вы `alert` до или после вызова `setWalk`?

<Solution>

Ваш `alert` должен выглядеть следующим образом:

<Sandpack>

```js
import { useState } from 'react';

export default function TrafficLight() {
  const [walk, setWalk] = useState(true);

  function handleClick() {
    setWalk(!walk);
    alert(walk ? 'Дальше Стоп' : 'Дальше Можно идти');
  }

  return (
    <>
      <button onClick={handleClick}>
        Изменить на {walk ? 'Стоп' : 'Можно идти'}
      </button>
      <h1 style={{
        color: walk ? 'darkgreen' : 'darkred'
      }}>
        {walk ? 'Можно идти' : 'Стоп'}
      </h1>
    </>
  );
}
```

```css
h1 { margin-top: 20px; }
```

</Sandpack>

На самом деле, нет никакой разницы, где вы разместите `setWalk`. Значение `walk` зафиксировано для этого рендера. Вызов `setWalk` изменит значение только для *следующего* рендера, но не повлияет на обработчик события для прошлого рендера.

Эта строка может показаться на первый взгляд нелогичной:

```js
alert(walk ? 'Дальше Стоп' : 'Дальше Можно идти');
```

В ней появится смысл, если читать её так: "Если светофор показывает 'Можно идти', в сообщении должно быть 'Дальше Стоп', иначе 'Дальше Можно идти'". Значение переменной `walk` внутри вашего обработчика событий соответствует значению `walk` этого рендера и не изменяется.

Вы можете убедиться в этом и провести подстановку необходимых значений. Когда `walk` равно `true`, вы получаете:

```js
<button onClick={() => {
  setWalk(false);
  alert('Дальше Стоп');
}}>
  Изменить на Стоп
</button>
<h1 style={{color: 'darkgreen'}}>
  Можно идти
</h1>
```

Таким образом, нажатие на "Изменить на Стоп" ставит в очередь рендер с `walk` равным `false` и оповещением "Дальше Стоп".

</Solution>

</Challenges>
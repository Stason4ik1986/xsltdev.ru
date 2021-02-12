# Перенаправление рефов

Перенаправление рефов позволяет автоматически передавать [реф](refs-and-the-dom.md) компонента одному из его дочерних элементов. Большинству компонентов перенаправление рефов не нужно, но оно может быть полезно, например, если вы пишете библиотеку. Рассмотрим наиболее частые сценарии.

## Перенаправление рефов в DOM-компоненты {#forwarding-refs-to-dom-components}

Допустим, у нас есть компонент `FancyButton`, который рендерит нативный DOM-элемент `button`:

```jsx
function FancyButton(props) {
  return (
    <button className="FancyButton">
      {props.children}
    </button>
  );
}
```

React-компоненты скрывают свои детали реализации, в том числе результат рендеринга. Реф элемента `button` из `FancyButton` **обычно и не требуется** другим компонентам. Это хорошо, поскольку такой подход не даёт компонентам излишне полагаться на структуру DOM друг друга.

Такая инкапсуляция хорошо подходит компонентам, которые описывают некую законченную часть приложения, например, `FeedStory` или `Comment`. А вот в «маленьких», часто повторно используемых компонентах, таких как `FancyButton` или `MyTextInput`, она может быть неудобной. Чтобы управлять фокусом, выделением и анимациями этих компонентов, придётся получить доступ к их DOM-узлам.

**Перенаправление рефов позволяет взять `ref` из атрибутов компонента, и передать («перенаправить») его одному из дочерних компонентов.**

В данном примере мы используем `React.forwardRef` в компоненте `FancyButton`, чтобы получить реф и передать его в дочерний DOM-элемент `button`.

```jsx
// highlight-range{1-2}
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// Теперь реф будет указывать непосредственно на DOM-узел button:
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

Таким образом, когда мы будем применять `FancyButton` в других компонентах, мы сможем получить реф находящегося в нём DOM-узла `button` и использовать его так же, как если бы мы рендерили непосредственно `button`.

Рассмотрим этот пример пошагово:

1. Мы создаём [реф](refs-and-the-dom.md), вызвав `React.createRef` и записываем его в переменную `ref`.
2. Мы передаём переменную `ref` в `<FancyButton ref={ref}>`, указывая её в JSX-атрибуте.
3. React передаёт `ref` в функцию `(props, ref) => ...` внутри `forwardRef` в качестве второго аргумента.
4. Мы передаём аргумент `ref` дальше в `<button ref={ref}>`, указывая его в JSX-атрибуте.
5. После привязки рефа `ref.current` будет указывать на DOM-узел `<button>`.

> Примечание
>
> Второй аргумент `ref` существует только в том случае, если вы создаёте компонент через функцию `React.forwardRef`. Обычные функциональные или классовые компоненты не получают `ref` в качестве аргумента или пропа.
>
> Перенаправить реф можно не только в DOM-компонент, но и в экземпляр классового компонента.

## Примечание для разработчиков библиотек компонентов {#note-for-component-library-maintainers}

**Если вы впервые использовали `forwardRef` в компоненте библиотеки, то следует сделать новую версию мажорной и указать на обратную несовместимость изменений.** Причина этого в том, что, скорее всего, компонент станет вести себя заметно иначе (например, изменится тип экспортируемых данных и элемент, к которому привязан реф), в результате чего приложения и другие библиотеки, полагающиеся на старое поведение, перестанут работать.

По этой же причине мы рекомендуем не вызывать `React.forwardRef` условно (то есть сперва проверяя, что эта функция определена). Это изменит поведение вашей библиотеки и приложения ваших пользователей могут перестать работать при обновлении самого React.

## Перенаправление рефов в компонентах высшего порядка {#forwarding-refs-in-higher-order-components}

Особенно полезным перенаправление может оказаться в [компонентах высшего порядка](higher-order-components.md) (также известных как HOC). Начнём с примера, в котором HOC выводит пропсы компонента в консоль:

```jsx
// highlight-next-line
function logProps(WrappedComponent) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('старые пропсы:', prevProps);
      console.log('новые пропсы:', this.props);
    }

    render() {
      // highlight-next-line
      return <WrappedComponent {...this.props} />;
    }
  }

  return LogProps;
}
```

Компонент высшего порядка `logProps` передаёт все пропсы в компонент, который он оборачивает, так что рендерить они будут одно и то же. С его помощью мы будем выводить в консоль все пропсы, переданные в наш компонент с кнопкой:

```jsx
class FancyButton extends React.Component {
  focus() {
    // ...
  }

  // ...
}

// Вместо экспорта FancyButton, мы экспортируем LogProps.
// Рендериться при этом будет FancyButton.
// highlight-next-line
export default logProps(FancyButton);
```

Обратите внимание, что в этом примере не будут передаваться рефы. Так происходит, потому что `ref` это не проп. Подобно `key`, React обрабатывает `ref` особым образом. Если вы укажите реф для HOC, он привяжется к ближайшему к корню контейнера, а не к переданному в HOC компоненту.

Следовательно, рефы, предназначенные для компонента `FancyButton`, окажутся привязанными к компоненту `LogProps`:

```jsx
import FancyButton from './FancyButton';

// highlight-next-line
const ref = React.createRef();

// Компонент FancyButton, который мы импортировали — это HOC LogProps.
// Несмотря на то, что рендерят они одно и то же,
// реф в данном случае будет указывать на LogProps, а не на сам FancyButton!
// Это значит, что мы не сможем, например, вызвать ref.current.focus()
// highlight-range{4}
<FancyButton
  label="Click Me"
  handleClick={handleClick}
  ref={ref}
/>;
```

К счастью, мы можем явно перенаправить рефы на компонент `FancyButton` внутри HOC при помощи API `React.forwardRef`. В `React.forwardRef` передаётся функция рендеринга, которая принимает аргументы `props` и `ref`, а возвращает узел React. Например:

```jsx
function logProps(Component) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }

    render() {
      // highlight-next-line
      const { forwardedRef, ...rest } = this.props;

      // Передаём в качестве рефа проп "forwardedRef"
      // highlight-next-line
      return <Component ref={forwardedRef} {...rest} />;
    }
  }

  // Обратите внимание, что React.forwardRef передает "ref" вторым аргументом.
  // Мы можем передать его дальше как проп, например, "forwardedRef",
  // а потом привязать его к компоненту.
  // highlight-range{1-3}
  return React.forwardRef((props, ref) => {
    return <LogProps {...props} forwardedRef={ref} />;
  });
}
```

## Изменение названия в инструментах разработки {#displaying-a-custom-name-in-devtools}

В `React.forwardRef` передаётся функция рендеринга. Эта функция определяет, как будет называться компонент в инструментах разработки.

Например, вот этот компонент будет называться «_ForwardRef_»:

```jsx
const WrappedComponent = React.forwardRef((props, ref) => {
  return <LogProps {...props} forwardedRef={ref} />;
});
```

Если присвоить имя функции рендеринга, то оно появится в названии компонента в инструментах разработки (например, «_ForwardRef(myFunction)_»):

```jsx
const WrappedComponent = React.forwardRef(
  function myFunction(props, ref) {
    return <LogProps {...props} forwardedRef={ref} />;
  }
);
```

Можно даже назначить функции свойство `displayName` и указать в нём, какой именно компонент обёрнут в HOC:

```jsx
function logProps(Component) {
  class LogProps extends React.Component {
    // ...
  }

  function forwardRef(props, ref) {
    return <LogProps {...props} forwardedRef={ref} />;
  }

  // Дадим компоненту более понятное имя в инструментах разработки.
  // Например, "ForwardRef(logProps(MyComponent))"
  // highlight-range{1-2}
  const name = Component.displayName || Component.name;
  forwardRef.displayName = `logProps(${name})`;

  return React.forwardRef(forwardRef);
}
```

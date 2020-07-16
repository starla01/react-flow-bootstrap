Este proyecto esta hecho con la base de [Create React App](https://github.com/facebook/create-react-app). y pretende explicar de manera sencilla como seguir el patron de `Flow` para desarrollar componentes mas seguros

## Componentes

### `Stateless Functional Components`

Además de las clases, React también admite componentes funcionales sin estado. Escribe estos componentes como escribiría una función:

```js
import * as React from "react";

type Props = {
  foo: number,
  bar?: string,
};

function MyComponent(props: Props) {
  props.doesNotExist; // Error! You did not define a `doesNotExist` prop.

  return <div>{props.bar}</div>;
}

<MyComponent foo={42} />;
```

### `Usando Default Props para componentes funcionales`

React también admite `defaultProps` en componentes funcionales sin estado.
De manera similar a los componentes de clase, los `defaultProps` para componentes funcionales sin estado funcionarán sin anotaciones de tipo adicionales.

```js
import * as React from "react";

type Props = {
  foo: number, // foo is required.
};

function MyComponent(props: Props) {}

MyComponent.defaultProps = {
  foo: 42, // ...but we have a default prop for foo.
};

// So we don't need to include foo.
<MyComponent />;
```

```
Nota: no es necesario que foo sea anulable en su propType.
Flow se asegurará de que foo sea opcional si tiene un accesorio predeterminado para foo.
```

# Manejo de eventos

### `Los tipos y las mejores prácticas para escribir controladores de eventos React con Flow`

En la sección ["Handling Events"](https://reactjs.org/docs/handling-events.html) de React docs, se proporcionan algunas recomendaciones diferentes sobre cómo definir los controladores de eventos. Si está utilizando Flow, le recomendamos que utilice la [sintaxis del inicializador de propiedades](https://babeljs.io/docs/en/babel-plugin-transform-class-properties/), ya que es la forma más fácil de escribir estáticamente. La sintaxis del inicializador de propiedades se ve así:

```js
class MyComponent extends React.Component<{}> {
  handleClick = (event) => {
    /* ... */
  };
}
```

Para escribir controladores de eventos, puede usar los tipos `SyntheticEvent<T>` como este:

```js
import * as React from "react";

class MyComponent extends React.Component<{}, { count: number }> {
  handleClick = (event: SyntheticEvent<HTMLButtonElement>) => {
    // To access your button instance use `event.currentTarget`.
    (event.currentTarget: HTMLButtonElement);

    this.setState((prevState) => ({
      count: prevState.count + 1,
    }));
  };

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.handleClick}>Increment</button>
      </div>
    );
  }
}
```

También hay tipos de eventos sintéticos más específicos como `SyntheticKeyboardEvent<T>`, `SyntheticMouseEvent<T>` o `SyntheticTouchEvent<T>`. Los tipos `SyntheticEvent<T>` toman todos un argumento de tipo único. El tipo de elemento HTML en el que se colocó el controlador de eventos.

Si no desea agregar el tipo de su instancia de elemento, también puede usar `SyntheticEvent` sin argumentos de tipo como este: `SyntheticEvent<>`.

`Nota: Para obtener la instancia del elemento, como HTMLButtonElement en el ejemplo anterior, es un error común usar event.target en lugar de event.currentTarget. La razón por la que desea usar event.currentTarget es que event.target puede ser el elemento incorrecto debido a la propagación de eventos.`

`Nota: React usa su propio sistema de eventos, por lo que es importante usar los tipos SyntheticEvent en lugar de los tipos DOM como Event, KeyboardEvent y MouseEvent.`

Los tipos `SyntheticEvent<T>` que React proporciona y los eventos DOM con los que están relacionados son:

`SyntheticEvent<T>` for [Event](https://developer.mozilla.org/en-US/docs/Web/API/Event)

`SyntheticAnimationEvent<T>` for [AnimationEvent](https://developer.mozilla.org/en-US/docs/Web/API/AnimationEvent)

`SyntheticCompositionEvent<T>` for [CompositionEvent](https://developer.mozilla.org/en-US/docs/Web/API/CompositionEvent)

`SyntheticInputEvent<T>` for [InputEvent](https://developer.mozilla.org/en-US/docs/Web/API/InputEvent)

`SyntheticUIEvent<T>` for [UIEvent](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent)

`SyntheticFocusEvent<T>` for [FocusEvent](https://developer.mozilla.org/en-US/docs/Web/API/FocusEvent)

`SyntheticKeyboardEvent<T>` for [KeyboardEvent](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent)

`SyntheticMouseEvent<T>` for [MouseEvent](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent)

`SyntheticDragEvent<T>` for [DragEvent](https://developer.mozilla.org/en-US/docs/Web/API/DragEvent)

`SyntheticWheelEvent<T>` for [WheelEvent](https://developer.mozilla.org/en-US/docs/Web/API/WheelEvent)

`SyntheticTouchEvent<T>` for [TouchEvent](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent)

`SyntheticTransitionEvent<T>` for [TransitionEvent](https://developer.mozilla.org/en-US/docs/Web/API/TransitionEvent)

# funciones de referencia

### `Cómo usar las funciones de referencia React tipificadas de Flow`

React le permite tomar la instancia de un elemento o componente con [funciones ref](https://facebook.github.io/react/docs/refs-and-the-dom.html). Para usar una función de referencia, agregue un tipo de instancia [quizás a su clase](https://flow.org/en/docs/types/maybe/) y asigne su instancia a esa propiedad en su función de referencia.

```js
import * as React from "react";

class MyComponent extends React.Component<{}> {
  // The `?` here is important because you may not always have the instance.
  button: ?HTMLButtonElement;

  render() {
    return <button ref={(button) => (this.button = button)}>Toggle</button>;
  }
}
```

Los `?` in `?HTMLButtonElement` es importante. En el ejemplo anterior, el primer argumento para `ref` será `HTMLButtonElement` | nulo como React [llamará a su devolución de llamada de referencia con nulo](https://facebook.github.io/react/docs/refs-and-the-dom.html#adding-a-ref-to-a-dom-element) cuando el componente se desmonte. Además, la propiedad del `button` en `MyComponent` no se establecerá hasta que `React` haya finalizado la representación. Hasta entonces, la referencia de su `button` no estará definida. Protéjase contra estos casos y use un `?` (como en `?HTMLButtonElement`) para protegerse de errores.

# Children

### `Aprenda a usar Flow para escribir estrictamente los elementos secundarios de sus componentes React`

Los elementos de `React` pueden tener cero, uno o muchos hijos. Poder escribir estos elementos secundarios con Flow le permite crear API expresivas con elementos secundarios React.

En general, el tipo que debe probar primero al agregar un tipo para los elementos secundarios de su componente React es React.Node.

Mira esto:

```js
import * as React from "react";

type Props = {
  children?: React.Node,
};

function MyComponent(props: Props) {
  return <div>{props.children}</div>;
}
```

`Nota: Debe usar import * como React from 'react' aquí en lugar de import React from 'react' para obtener acceso al tipo React.Node. Explicamos por qué eso está en la Referencia de tipo de React.`

Sin embargo, si desea hacer algo más poderoso con la API React children, necesitará una fuerte intuición de cómo React trata a los `children`. Veamos un par de casos antes de continuar ayudando a construir esa intuición.

Si ya tiene una fuerte intuición sobre cómo funcionan los `children` de React, no dude en pasar a nuestros ejemplos que demuestran cómo escribir varios patrones de `children` que comúnmente aparecen en los componentes de React.

Nuestro primer caso es un elemento sin `children`:

```js
<MyComponent />;

// which is the same as...
React.createElement(MyComponent, {});
```

Si no pasa ningún elemento secundario al crear un elemento de `MyComponent`, no se configurará `props.children`. Si intentas acceder a `props.children`, será `undefined`.

¿Qué pasa cuando tienes solo un `children`?

```js
<MyComponent>{42}</MyComponent>;

// which is the same as...
React.createElement(MyComponent, {}, 42);
```

Si pasa un valor único, entonces `props.children` será exactamente ese valor único. Aquí, `props.children` será el número 42. Es importante destacar que `props.children` no será una matriz. Será exactamente el número 42.

Que pasa cuando tienes multiples `children`?

```js
<MyComponent>
  {1}
  {2}
</MyComponent>;

// which is the same as...
React.createElement(MyComponent, {}, 1, 2);
```

Ahora, si pasa dos valores, `props.children` será una matriz. Específicamente en este caso, los `children` serán [1, 2].

Varios `children` también pueden verse así:

```js
<MyComponent>{"hello"} world</MyComponent>;

// which is the same as...
React.createElement(MyComponent, {}, "hello", " world");
```

ó

```jsx
<MyComponent>
  {"hello"} <strong>world</strong>
</MyComponent>;

// which is the same as...
React.createElement(
  MyComponent,
  {},
  "hello",
  " ",
  React.createElement("strong", {}, "world")
);
```

En el que `props.children` sería la matriz ['hola', '', `<strong>mundo</strong>`].

Pasando al siguiente caso. ¿Qué sucede si tenemos un solo hijo, pero ese hijo es un `array`?

```js
<MyComponent>{[1, 2]}</MyComponent>;

// which is the same as...
React.createElement(MyComponent, {}, [1, 2]);
```

Esto sigue la misma regla que cuando pasas a un solo `children`, entonces los niños serán exactamente ese valor. Aunque [1, 2] es una matriz, es un valor único y, por lo tanto, `props.children` será exactamente ese valor. Es decir, `props.children` será un array [1, 2] y no un `array` de `arrays`.

Este caso ocurre a menudo cuando usa array.map () como en:

```js
<MyComponent>
  {messages.map((message) => (
    <strong>{message}</strong>
  ))}
</MyComponent>;

// which is the same as...
React.createElement(
  MyComponent,
  {},
  messages.map((message) => React.createElement("strong", {}, message))
);
```

Entonces, un solo elemento secundario se deja solo, pero ¿qué sucede si tenemos varios elementos secundarios que son conjuntos?

```js
<MyComponent>
  {[1, 2]}
  {[3, 4]}
</MyComponent>;

// which is the same as...
React.createElement(MyComponent, {}, [1, 2], [3, 4]);
```

Aquí `props.children` será una serie de `arrays`. Específicamente, los `children` serán [[1, 2], [3, 4]].

La regla para recordar con React `children` es que si no tiene `children`, entonces no se establecerá `props.children`, si tiene un solo hijo, `props.children` se establecerá exactamente en ese valor, y si tiene dos o más hijos, entonces `props.children` será una nueva gama de esos valores.

Ahora veamos cómo tomarías esta intuición y escribirías los `children` de varios componentes React.

Solo se permite un tipo de elemento específico como elementos secundarios.
A veces solo quieres un componente específico como hijos de tu componente React. Esto sucede a menudo cuando está creando un componente de tabla que necesita componentes secundarios de columna específicos, o una barra de pestañas que necesita una configuración específica para cada pestaña. Uno de esos componentes de la barra de pestañas que usa este patrón es el componente `<TabBarIOS>` de React Native.

El componente `<TabBarIOS>` de React Native solo permite elementos secundarios React y esos elementos deben tener un tipo de componente de `<TabBarIOS.Item>`. Se espera que use `<TabBarIOS>` como:

```js
<TabBarIOS>
  <TabBarIOS.Item>{/* ... */}</TabBarIOS.Item>
  <TabBarIOS.Item>{/* ... */}</TabBarIOS.Item>
  <TabBarIOS.Item>{/* ... */}</TabBarIOS.Item>
</TabBarIOS>
```

No puede hacer lo siguiente cuando usa `<TabBarIOS>`:

```jsx
<TabBarIOS>
  <TabBarIOS.Item>{/* ... */}</TabBarIOS.Item>
  <TabBarIOS.Item>{/* ... */}</TabBarIOS.Item>
  <View>{/* ... */}</View>
  <SomeOtherComponent>{/* ... */}</SomeOtherComponent>
  <TabBarIOS.Item>{/* ... */}</TabBarIOS.Item>
</TabBarIOS>
```

¿Ves cómo agregamos `<View>` y `<SomeOtherComponent>` como elementos secundarios a `<TabBarIOS>`? Esto no está permitido y `<TabBarIOS>` arrojará un error. ¿Cómo nos aseguramos de que Flow no permita este patrón?

```jsx
import * as React from "react";

class TabBarIOSItem extends React.Component<{}> {
  // implementation...
}

type Props = {
  children: React.ChildrenArray<React.Element<typeof TabBarIOSItem>>,
};

class TabBarIOS extends React.Component<Props> {
  static Item = TabBarIOSItem;
  // implementation...
}

<TabBarIOS>
  <TabBarIOS.Item>{/* ... */}</TabBarIOS.Item>
  <TabBarIOS.Item>{/* ... */}</TabBarIOS.Item>
  <TabBarIOS.Item>{/* ... */}</TabBarIOS.Item>
</TabBarIOS>;
```

Establecemos el tipo de `props` en `React.ChildrenArray <React.Element <typeof TabBarIOSItem >>`, lo que garantizará que <TabBarIOS> solo debe tener elementos secundarios que sean elementos TabBarIOS.Item React.

Nuestra referencia de tipos tiene más información sobre `React.ChildrenArray<T>` y `React.Element<typeof Component>`.

`Nota:` Si desea métodos como map() y forEach() o para manejar un React.ChildrenArray<T> como una matriz de JavaScript normal, entonces React proporciona la [API React.Children](https://facebook.github.io/react/docs/react-api.html#react.children) para hacer esto. Tiene funciones como `React.Children.toArray(props.children)` que puede usar para tratar su `React.ChildrenArray<T>` como una matriz plana (flat array).

## Hacer cumplir que un componente solo obtenga un solo `children`.

A veces, desea exigir que su componente solo reciba un solo `children`. Puede usar la función `React.Children.only()` para aplicar esta restricción, pero también puede aplicar esto en Flow. Para hacer esto, en lugar de ajustar el tipo para sus hijos en `React.ChildrenArray<T>`, especifique un argumento de elemento único, así:

```js
import * as React from "react";

type Props = {
  children: React.Element<any>,
};

function MyComponent(props: Props) {
  // implementation...
}

// Not allowed! You must have children.
<MyComponent />;

// Not ok! We have multiple element children.
<MyComponent>
  <div />
  <div />
  <div />
</MyComponent>;

// This is ok. We have a single element child.
<MyComponent>
  <div />
</MyComponent>;
```

Función de escritura para `children` u otros `child types` exóticos.

React le permite pasar cualquier valor como `children` de un componente `React`. Hay algunos usos creativos de esta capacidad, como el uso de una función para `children` que podría verse así:

```js
<MyComponent>{(data) => <div>{data.foo}</div>}</MyComponent>
```

`react-router versión 4` solicita una función como `children` a su componente <Route>. Debería proporcionar una función como hijos para `react-router` de esta manera:

```js
<Route path={to}>
  {({ match }) => (
    <li className={match ? "active" : ""}>
      <Link to={to} {...rest} />
    </li>
  )}
</Route>
```

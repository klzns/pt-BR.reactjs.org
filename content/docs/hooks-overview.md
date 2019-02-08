---
id: hooks-overview
title: Hooks at a Glance
permalink: docs/hooks-overview.html
next: hooks-state.html
prev: hooks-intro.html
---

Os *Hooks* são uma nova funcionalidade disponível em React 16.8. Eles permitem usar state e outras funcionalidades de React sem ter que escrever classes.

Hooks são [backwards-compatible](/docs/hooks-intro.html#no-breaking-changes). Essa página descreve uma visão geral de Hooks para usuário de React experientes. É uma perspectiva direta. Se ficar confuso, procure por uma caixa amarela como esta:

>Explicação detalhada
>
>Leia em [Motivação](/docs/hooks-intro.html#motivation) para aprender por quê nós estamos introduzindo Hooks em React.

**↑↑↑ Toda seção termina com uma caixa amarela como esta.** Elas linkam para explicações detalhadas.

## 📌 State Hook {#-state-hook}

Esse exemplo renderiza um contador. Quando você clica em um botão, ele incrementa o valor:

```js{1,4,5}
import React, { useState } from 'react';

function Example() {
  // Declara uma nova variável com state, vamos chamá-la de "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click here
      </button>
    </div>
  );
}
```

Aqui, `useState` é um *Hook* (iremos falar mais sobre isso em breve). Nós chamamos ele dentro de um componente de função para adicionar um state local a ele. React irá preservar o state entre re-renderizações. `useState` retorna um par: o state *atual* e uma função que permite você atualizá-lo. Você pode chamar essa função de um event handler ou de outro lugar. É parecido com `this.setState` em uma classe, exceto que ele não combina o state antigo com o novo.(Nós vamos mostrar um exemplo comparando `useState` com `this.state` em [Usando o State Hook](/docs/hooks-state.html).)

O único argumento para `useState` é o state inicial. No exemplo acima, é `0` porque o nosso contador começa com zero. Note que diferente de `this.state`, o state aqui não tem que ser um objeto, mas pode ser se quiser. O argumento de state inicial só é usado durante a primeira renderização.

#### Declarando múltiplas variáveis de state {#declaring-multiple-state-variables}

Você pode usar o State Hook mais de uma vez em um único componente:

```js
function ExampleWithManyStates() {
  // Declare múltiplas variáveis state!
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
  // ...
}
```

A sintaxe de [desestruturação de array](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Operators/Atribuicao_via_desestruturacao#Desestrutura%C3%A7%C3%A3o_de_array) permite-nos dar diferentes nomes para as variáveis de state que declaramos chamando `useState`. Estes nomes não são parte da API de `useState`. Em vez disso, React assume que se você está chamando `useState` várias vezes, você faça na mesma ordem durante todas as renderizações. Nós iremos voltar no por quê isso funciona e quando isso vai ser útil depois.

#### Mas o que é um Hook? {#but-what-is-a-hook}

Hooks são funções que permitem você "se enganchar" no state React e ciclo de vida de componentes de função. Hooks não funcionam dentro de classes -- eles permitem usar React sem classes. (Nós [não recomendamos](/docs/hooks-intro.html#gradual-adoption-strategy) reescrever seus componentes existentes da noite pro dia, mas você pode começar a usar Hooks nos novos se você quiser.)

React provê alguns Hooks nativos como `useState`. Você tambem pode criar seus próprios Hooks para reutilizar comportamentos que usam estado entre componentes. Primeiro, iremos começar a ver os Hooks nativos.

>Explicação detalhada
>
>Você pode aprender mais sobre o State Hooke em uma página dedicada: [Usando State Hook](/docs/hooks-state.html).

## ⚡️ Effect Hook {#️-effect-hook}

Você provavelmente já pegou dados, fez subscriptions, ou manualmente alterou o DOM a partir de componentes React antes. Nós chamamos essas operações de "efeitos colaterais" (ou "efeitos" encurtando) porque eles podem afetar outros componentes e não podem ser feitos durante a renderização.

O Effect Hook, `useEffect`, adiciona a habilidade de fazer efeitos colaterais a partir de um componente de função. Isso tem o mesmo propósito de `componentDidMount`, `componentDidUpdate`, e `componentWillUnmount` em classes React, mas unificados em uma única API. (Nós iremos mostrar exemplos comparando `useEffect` para esses métodos em [Usando o Effect Hook](/docs/hooks-effect.html).)

Por exemplo, este componente define o título da página após o React atualizar o DOM:

```js{1,6-10}
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // Parecido com componentDidMount e componentDidUpdate:
  useEffect(() => {
    // Atualiza o título da página usando a API do browser
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

Quando você chama `useEffect`, você está dizendo ao React para rodar sua função de "efeito" depois de fazer as mudanças no DOM. Efeitos são declarados dentro do componente para que ele tenha acesso as props e ao state. Por padrão, React roda os efeitos depois de cada render -- *incluindo* o primeiro render. (Nós iremos falar mais como isso se compara aos lifecycles de classe em [Usando o Effect Hook](/docs/hooks-effect.html).)

Efeitos podem também opcionalmente especificar como "limpá-los" retornando uma função. Por exemplo, esse componente usa um efeito para subscribe o status online de um amigo, e limpa fazendo unsubscribe:

```js{10-16}
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);

    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

Neste exemplo, React faria o unsubscribe de nossa `ChatAPI` quando o componente faz unmount, e antes de re-rodar o efeito devido a um render subsequente. (Se quiser, tem outra forma de [dizer ao React para pular o re-subscribe](/docs/hooks-effect.html#tip-optimizing-performance-by-skipping-effects) se a `props.friend.id` passada para `ChatAPI` não mudou.)

Assim como `useState`, você pode usar mais de um efeito em um componente:

```js{3,8}
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }
  // ...
```

Hooks permite você organizar seus efeitos colaterais em um componente por quais pedaços estão relacionados (como adicionar ou remover uma subscription), ao invés de forçar a separar baseado nos métodos de ciclo de vida.

>Explicação detalhada
>
>Você pode aprender mais sobre `useEffect` em uma página dedicada: [Usando o Effect Hook](/docs/hooks-effect.html).

## ✌️ Regra dos Hooks {#️-rules-of-hooks}

Hooks são funções JavaScript, mas elas impõem duas regras adicionais:

* Only call Hooks **at the top level**. Don’t call Hooks inside loops, conditions, or nested functions.
* Only call Hooks **from React function components**. Don’t call Hooks from regular JavaScript functions. (There is just one other valid place to call Hooks -- your own custom Hooks. We'll learn about them in a moment.)

We provide a [linter plugin](https://www.npmjs.com/package/eslint-plugin-react-hooks) to enforce these rules automatically. We understand these rules might seem limiting or confusing at first, but they are essential to making Hooks work well.

>Detailed Explanation
>
>You can learn more about these rules on a dedicated page: [Rules of Hooks](/docs/hooks-rules.html).

## 💡 Building Your Own Hooks {#-building-your-own-hooks}

Sometimes, we want to reuse some stateful logic between components. Traditionally, there were two popular solutions to this problem: [higher-order components](/docs/higher-order-components.html) and [render props](/docs/render-props.html). Custom Hooks let you do this, but without adding more components to your tree.

Earlier on this page, we introduced a `FriendStatus` component that calls the `useState` and `useEffect` Hooks to subscribe to a friend's online status. Let's say we also want to reuse this subscription logic in another component.

First, we'll extract this logic into a custom Hook called `useFriendStatus`:

```js{3}
import React, { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

It takes `friendID` as an argument, and returns whether our friend is online.

Now we can use it from both components:


```js{2}
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

```js{2}
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

The state of these components is completely independent. Hooks are a way to reuse *stateful logic*, not state itself. In fact, each *call* to a Hook has a completely isolated state -- so you can even use the same custom Hook twice in one component.

Custom Hooks are more of a convention than a feature. If a function's name starts with "`use`" and it calls other Hooks, we say it is a custom Hook. The `useSomething` naming convention is how our linter plugin is able to find bugs in the code using Hooks.

You can write custom Hooks that cover a wide range of use cases like form handling, animation, declarative subscriptions, timers, and probably many more we haven't considered. We are excited to see what custom Hooks the React community will come up with.

>Detailed Explanation
>
>You can learn more about custom Hooks on a dedicated page: [Building Your Own Hooks](/docs/hooks-custom.html).

## 🔌 Other Hooks {#-other-hooks}

There are a few less commonly used built-in Hooks that you might find useful. For example, [`useContext`](/docs/hooks-reference.html#usecontext) lets you subscribe to React context without introducing nesting:

```js{2,3}
function Example() {
  const locale = useContext(LocaleContext);
  const theme = useContext(ThemeContext);
  // ...
}
```

And [`useReducer`](/docs/hooks-reference.html#usereducer) lets you manage local state of complex components with a reducer:

```js{2}
function Todos() {
  const [todos, dispatch] = useReducer(todosReducer);
  // ...
```

>Detailed Explanation
>
>You can learn more about all the built-in Hooks on a dedicated page: [Hooks API Reference](/docs/hooks-reference.html).

## Next Steps {#next-steps}

Phew, that was fast! If some things didn't quite make sense or you'd like to learn more in detail, you can read the next pages, starting with the [State Hook](/docs/hooks-state.html) documentation.

You can also check out the [Hooks API reference](/docs/hooks-reference.html) and the [Hooks FAQ](/docs/hooks-faq.html).

Finally, don't miss the [introduction page](/docs/hooks-intro.html) which explains *why* we're adding Hooks and how we'll start using them side by side with classes -- without rewriting our apps.

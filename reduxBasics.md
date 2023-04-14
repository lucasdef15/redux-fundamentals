# O Básico sobre Redux

Vamos dar uma olhada rápida nas peças que compõem um aplicativo Redux e como ele funciona.

# O Redux Store

O centro de todo aplicativo Redux é a "store". Uma "store" é um contêiner que armazena o estado global do seu aplicativo.

Um "store" é um objeto JavaScript com algumas funções e habilidades que o diferenciam de um objeto global comum:

- É importante nunca modificar ou mudar o estado que está sendo mantido dentro do Redux store diretamente.
- Em vez disso, a única maneira de atualizar o estado é criando um objeto "action" que descreva "algo que aconteceu na aplicação" e, em seguida, enviando essa "action" para a "store".
- Quando uma "action" é enviada, a "store" executa uma função reducer raiz e calcula o novo estado com base no estado anterior e na "action".
- Por fim, quando a "store" notifica os "subscribers" que o estado foi atualizado, isso significa que os componentes da interface do usuário (UI) que se inscreveram na "store" serão notificados da mudança e poderão se atualizar com os novos dados. Isso garante que a UI sempre reflita o estado mais recente da aplicação, sem modificar diretamente o estado mantido dentro da "store" do Redux.

```html
<html>
  <head>
    <title>Redux basic example</title>
    <script src="https://unpkg.com/redux@latest/dist/redux.min.js"></script>
  </head>
  <body>
    <div>
      <p>
        Clicked: <span id="value">0</span> times
        <button id="increment">+</button>
        <button id="decrement">-</button>
        <button id="incrementIfOdd">Increment if odd</button>
        <button id="incrementAsync">Increment async</button>
      </p>
    </div>
    <script>
      // Define an initial state value for the app
      const initialState = {
        value: 0,
      };

      // Create a "reducer" function that determines what the new state
      // should be when something happens in the app
      function counterReducer(state = initialState, action) {
        // Reducers usually look at the type of action that happened
        // to decide how to update the state
        switch (action.type) {
          case "counter/incremented":
            return { ...state, value: state.value + 1 };
          case "counter/decremented":
            return { ...state, value: state.value - 1 };
          default:
            // If the reducer doesn't care about this action type,
            // return the existing state unchanged
            return state;
        }
      }

      // Create a new Redux store with the `createStore` function,
      // and use the `counterReducer` for the update logic
      const store = Redux.createStore(counterReducer);

      // Our "user interface" is some text in a single HTML element
      const valueEl = document.getElementById("value");

      // Whenever the store state changes, update the UI by
      // reading the latest store state and showing new data
      function render() {
        const state = store.getState();
        valueEl.innerHTML = state.value.toString();
      }

      // Update the UI with the initial data
      render();
      // And subscribe to redraw whenever the data changes in the future
      store.subscribe(render);

      // Handle user inputs by "dispatching" action objects,
      // which should describe "what happened" in the app
      document
        .getElementById("increment")
        .addEventListener("click", function () {
          store.dispatch({ type: "counter/incremented" });
        });

      document
        .getElementById("decrement")
        .addEventListener("click", function () {
          store.dispatch({ type: "counter/decremented" });
        });

      document
        .getElementById("incrementIfOdd")
        .addEventListener("click", function () {
          // We can write logic to decide what to do based on the state
          if (store.getState().value % 2 !== 0) {
            store.dispatch({ type: "counter/incremented" });
          }
        });

      document
        .getElementById("incrementAsync")
        .addEventListener("click", function () {
          // We can also write async logic that interacts with the store
          setTimeout(function () {
            store.dispatch({ type: "counter/incremented" });
          }, 1000);
        });
    </script>
  </body>
</html>
```

Como o Redux é uma biblioteca JS autônoma, sem dependências, este exemplo é escrito carregando apenas uma única tag de script para a biblioteca Redux e usando JS e HTML básicos para a interface do usuário. Na prática, o Redux é normalmente usado instalando os pacotes Redux do NPM, e a interface do usuário é criada usando uma biblioteca como o React.

Vamos dividir este exemplo em partes para que possamos entender melhor o que está acontecendo.

# State, Actions, e Reducers

Começamos definindo um valor para o estado inicial que descreve a aplicação:

```js
// Define an initial state value for the app
const initialState = {
  value: 0,
};
```

Para este aplicativo, vamos acompanhar um único número com o valor atual do nosso contador.

Os aplicativos Redux normalmente têm um objeto JS como a peça raiz do estado, com outros valores dentro desse objeto.

Em seguida, definimos uma função reducer. O reducer recebe dois argumentos, o estado atual e um objeto de ação descrevendo o que aconteceu. Quando o aplicativo Redux é iniciado, ainda não temos nenhum estado, portanto, fornecemos o **initialState** como o valor padrão para este reducer:

```js
// Create a "reducer" function that determines what the new state
// should be when something happens in the app
function counterReducer(state = initialState, action) {
  // Reducers usually look at the type of action that happened
  // to decide how to update the state
  switch (action.type) {
    case "counter/incremented":
      return { ...state, value: state.value + 1 };
    case "counter/decremented":
      return { ...state, value: state.value - 1 };
    default:
      // If the reducer doesn't care about this action type,
      // return the existing state unchanged
      return state;
  }
}
```

Os objetos "action"(ação) sempre têm um campo "type"(tipo), que é uma string que você fornece, e atua como um nome exclusivo para a ação. O "type" deve ser um nome legível para que qualquer pessoa que olhe para este código entenda o que significa. Neste caso, usamos a palavra "counter" como a primeira metade do nosso tipo de ação, e a segunda metade é uma descrição do que aconteceu. Neste caso, nosso "contador" foi "incrementado", então escrevemos o tipo de ação como "counter/incremented".

Com base no tipo de ação, precisamos retornar um novo objeto para ser o novo resultado de estado, ou retornar o objeto de estado existente se nada deve ser alterado. Note que atualizamos o estado de forma imutável, copiando o estado existente e atualizando a cópia, em vez de modificar diretamente o objeto original.

# Store

Agora que temos uma função de reducer, podemos criar uma instância de "store" chamando a API **createStore** da biblioteca Redux.

```js
// Create a new Redux store with the `createStore` function,
// and use the `counterReducer` for the update logic
const store = Redux.createStore(counterReducer);
```

Passamos a função reducer para o **createStore**, que usa a função reducer para gerar o estado inicial e calcular futuras atualizações.

# UI

Na interface do usuário de qualquer aplicativo, o estado existente é mostrado na tela. Quando um usuário faz algo, o aplicativo atualiza seus dados e, em seguida, renderiza a interface do usuário com esses valores.

```js
// Our "user interface" is some text in a single HTML element
const valueEl = document.getElementById("value");

// Whenever the store state changes, update the UI by
// reading the latest store state and showing new data
function render() {
  const state = store.getState();
  valueEl.innerHTML = state.value.toString();
}

// Update the UI with the initial data
render();
// And subscribe to redraw whenever the data changes in the future
store.subscribe(render);
```

Neste pequeno exemplo, estamos usando apenas alguns elementos HTML básicos como nossa interface do usuário (UI), com um único elemento <div> mostrando o valor atual.

Então, escrevemos uma função que sabe como obter o estado mais recente do "Redux Store" usando o método **store.getState()**, em seguida, usa esse valor e atualiza a UI para mostrá-lo.

O "Redux Store" nos permite chamar **store.subscribe()** e passar uma função "subscriber callback" que será chamada sempre que o "store" for atualizado. Assim, podemos passar nossa função de renderização como o "subscriber"(assinante) e saber que sempre que o "store" for atualizado, podemos atualizar a UI com o último valor.

O Redux em si é uma biblioteca independente que pode ser usada em qualquer lugar. Isso também significa que ele pode ser usado com qualquer camada de interface do usuário.

# Dispatching(enviando) Actions

Por último, precisamos responder à entrada do usuário criando objetos "action" que descrevam o que aconteceu e "dispatching"(despachando-os) para "store". Quando chamamos store.dispatch(action), "store" executa o "reducer", calcula o estado atualizado e executa os assinantes para atualizar a interface do usuário.

```js
// Handle user inputs by "dispatching" action objects,
// which should describe "what happened" in the app
document.getElementById("increment").addEventListener("click", function () {
  store.dispatch({ type: "counter/incremented" });
});

document.getElementById("decrement").addEventListener("click", function () {
  store.dispatch({ type: "counter/decremented" });
});

document
  .getElementById("incrementIfOdd")
  .addEventListener("click", function () {
    // We can write logic to decide what to do based on the state
    if (store.getState().value % 2 !== 0) {
      store.dispatch({ type: "counter/incremented" });
    }
  });

document
  .getElementById("incrementAsync")
  .addEventListener("click", function () {
    // We can also write async logic that interacts with the store
    setTimeout(function () {
      store.dispatch({ type: "counter/incremented" });
    }, 1000);
  });
```

Aqui, vamos despachar as ações que farão o redutor adicionar 1 ou subtrair 1 do valor atual do contador.

Também podemos escrever código que despacha uma ação somente se uma determinada condição for verdadeira, ou escrever algum código assíncrono que despacha uma ação após um tempo.

# Data Flow

Podemos resumir o fluxo de dados em um aplicativo Redux com este diagrama. Ele representa como:

- As ações são despachadas em resposta a uma interação do usuário, como um clique.
- O store executa a função do reducer para calcular um novo estado.
- A interface do usuário lê o novo estado para exibir os novos valores.

(Não se preocupe se essas peças ainda não estão claras! Mantenha essa imagem em mente ao seguir o resto deste dos tutorias, e você verá como as peças se encaixam.)

![Alt text](./assets/images/reduxdataflowdiagram.gif)

# Le Direttive Personalizzate {#custom-directives}

<script setup>
const vHighlight = {
  mounted: el => {
    el.classList.add('is-highlight')
  }
}
</script>

<style>
.vt-doc p.is-highlight {
  margin-bottom: 0;
}

.is-highlight {
  background-color: yellow;
  color: black;
}
</style>

<style>
.vt-doc p.is-highlight {
  margin-bottom: 0;
}

.is-highlight {
  background-color: yellow;
  color: black;
}
</style>

## Introduzione {#introduction}

Oltre all'insieme di direttive predefinite incluse nel core (come `v-model` o `v-show`), Vue ti permette anche di registrare le tue direttive personalizzate.

Abbiamo introdotto due forme di riutilizzo del codice in Vue: i [componenti](/guide/essentials/component-basics) e i [composables](./composables). I componenti sono i principali mattoni di costruzione, mentre i composables sono incentrati sul riutilizzo della logica con stato. Le direttive personalizzate, invece, sono principalmente destinate al riutilizzo della logica che coinvolge l'accesso al DOM di basso livello su elementi semplici.

A custom directive is defined as an object containing lifecycle hooks similar to those of a component. The hooks receive the element the directive is bound to. Here is an example of a directive that adds a class to an element when it is inserted into the DOM by Vue:

<div class="composition-api">

```vue
<script setup>
// enables v-highlight in templates
const vHighlight = {
  mounted: (el) => {
    el.classList.add('is-highlight')
  }
}
</script>

## Introduction {#introduction}

In addition to the default set of directives shipped in core (like `v-model` or `v-show`), Vue also allows you to register your own custom directives.

We have introduced two forms of code reuse in Vue: [components](/guide/essentials/component-basics) and [composables](./composables). Components are the main building blocks, while composables are focused on reusing stateful logic. Custom directives, on the other hand, are mainly intended for reusing logic that involves low-level DOM access on plain elements.

Una direttiva personalizzata è definita come un oggetto contenente degli hook del ciclo di vita simili a quelli di un componente. Gli hook ricevono l'elemento a cui è legata la direttiva. Ecco un esempio di una direttica che aggiunge una classe ad un elemento quando è inserito nel DOM da Vue:

<div class="composition-api">

```vue
<script setup>
// enables v-highlight in templates
const vHighlight = {
  mounted: (el) => {
    el.classList.add('is-highlight')
  }
}
</script>

<template>
  <p v-highlight>This sentence is important!</p>
</template>
```

</div>

<div class="options-api">

```js
const highlight = {
  mounted: (el) => el.classList.add('is-highlight')
}

export default {
  directives: {
    // enables v-highlight in template
    highlight
  }
}
```

```vue-html
<p v-highlight>This sentence is important!</p>
```

</div>

<div class="demo">
  <p v-highlight>This sentence is important!</p>
</div>

<div class="composition-api">

In `<script setup>`, any camelCase variable that starts with the `v` prefix can be used as a custom directive. In the example above, `vHighlight` can be used in the template as `v-highlight`.

If you are not using `<script setup>`, custom directives can be registered using the `directives` option:

```js
export default {
  setup() {
    /*...*/
  },
  directives: {
    // enables v-highlight in template
    highlight: {
      /* ... */
    }
  }
}
```

</div>

<div class="options-api">

Similar to components, custom directives must be registered so that they can be used in templates. In the example above, we are using local registration via the `directives` option.

</div>

It is also common to globally register custom directives at the app level:

```js
const app = createApp({})

// make v-highlight usable in all components
app.directive('highlight', {
  /* ... */
})
```

It is possible to type global custom directives by extending the `GlobalDirectives` interface from `vue`

More Details: [Typing Custom Global Directives](/guide/typescript/composition-api#typing-global-custom-directives) <sup class="vt-badge ts" />

## When to use custom directives {#when-to-use}

Custom directives should only be used when the desired functionality can only be achieved via direct DOM manipulation.

A common example of this is a `v-focus` custom directive that brings an element into focus.

<div class="composition-api">

```vue
<script setup>
// abilita v-focus nei template
const vFocus = {
  mounted: (el) => el.focus()
}
</script>

<template>
  <input v-focus />
</template>
```

</div>

<div class="options-api">

```js
const focus = {
  mounted: (el) => el.focus()
}

export default {
  directives: {
    // abilita v-focus nei template
    focus
  }
}
```

```vue-html
<input v-focus />
```

</div>

This directive is more useful than the `autofocus` attribute because it works not just on page load - it also works when the element is dynamically inserted by Vue!

Declarative templating with built-in directives such as `v-bind` is recommended when possible because they are more efficient and server-rendering friendly.

## Directive Hooks {#directive-hooks}

Un oggetto che definisce la direttiva può fornire diverse funzioni hook (tutte opzionali):

```js
const myDirective = {
  // Chiamato prima dell'applicazione degli attributi o 
  // dei listener di eventi all'elemento a cui è legato
  created(el, binding, vnode) {
    // vedi sotto per i dettagli sugli argomenti
  },
  // chiamato subito prima che l'elemento venga inserito nel DOM.
  beforeMount(el, binding, vnode) {},
  // chiamato quando il componente genitore dell'elemento a cui è legato
  // e tutti i suoi figli sono montati.
  mounted(el, binding, vnode) {},
  // chiamato prima che il componente genitore venga aggiornato
  beforeUpdate(el, binding, vnode, prevVnode) {},
  // chiamato dopo che il componente genitore e
  // tutti i suoi figli sono stati aggiornati
  updated(el, binding, vnode, prevVnode) {},
  // chiamato prima che il componente genitore venga smontato
  beforeUnmount(el, binding, vnode) {},
  // chiamato quando il componente genitore viene smontato
  unmounted(el, binding, vnode) {}
}
```

### Argomenti degli Hook {#hook-arguments}

Gli hook delle direttive ricevono questi argomenti:

- `el`: l'elemento al quale la direttiva è legata. Questo può essere utilizzato per manipolare direttamente il DOM.

- `binding`: un oggetto contenente le seguenti proprietà.

  - `value`: Il valore passato alla direttiva. Ad esempio in `v-my-directive="1 + 1"`, il valore sarebbe `2`.
  - `oldValue`: : Il valore precedente, disponibile solo in `beforeUpdate` e `updated`. È disponibile indipendentemente dal fatto che il valore sia cambiato o meno.
  - `arg`: L'argomento passato alla direttiva, se presente. Ad esempio in `v-my-directive:foo`, l'arg sarebbe `"foo"`.
  - `modifiers`: Un oggetto contenente i modificatori, se presenti. Ad esempio in `v-my-directive.foo.bar`, l'oggetto dei modificatori sarebbe `{ foo: true, bar: true }`.
  - `instance`: L'istanza del componente dove viene utilizzata la direttiva.
  - `dir`: l'oggetto di definizione della direttiva.

- `vnode`: il VNode sottostante che rappresenta l'elemento a cui è legato.
- `prevVnode`:  il VNode che rappresenta l'elemento a cui è legato dal render precedente. Disponibile solo negli hook `beforeUpdate` e `updated`.

Come esempio, considera il seguente uso della direttiva:

```vue-html
<div v-example:foo.bar="baz">
```

L'argomento `binding`  sarebbe un oggetto nella forma di:

```js
{
  arg: 'foo',
  modifiers: { bar: true },
  value: /* valore di `baz` */,
  oldValue: /* valore di `baz` dall'aggiornamento precedente */
}
```

In modo simile alle direttive native, gli argomenti delle direttive personalizzate possono essere dinamici. Ad esempio:

```vue-html
<div v-example:[arg]="value"></div>
```

Qui l'argomento della direttiva verrà aggiornato in maniera reattiva in base alla proprietà arg nello stato del nostro componente.

:::tip Nota
A parte `el`, dovresti trattare questi argomenti come di sola lettura e non modificarli mai. Se hai bisogno di condividere informazioni attraverso gli hook, è consigliato farlo tramite il [dataset](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset) dell'elemento.
:::

## Funzione Shorthand {#function-shorthand}

È comune che una direttiva personalizzata abbia lo stesso comportamento per `mounted` e `updated`, senza bisogno degli altri hook. In tali casi possiamo definire la direttiva come una funzione:

```vue-html
<div v-color="color"></div>
```

```js
app.directive('color', (el, binding) => {
  // Questo verrà chiamato sia per `mounted` che per `updated`
  el.style.color = binding.value
})
```

## Oggetti Literal {#object-literals}

Se la tua direttiva ha bisogno di valori multipli, puoi passare anche un oggetto literal JavaScript. Ricorda che le direttive possono accettare qualsiasi espressione JavaScript valida.

```vue-html
<div v-demo="{ color: 'white', text: 'ciao!' }"></div>
```

```js
app.directive('demo', (el, binding) => {
  console.log(binding.value.color) // => "white"
  console.log(binding.value.text) // => "ciao!"
})
```

## Utilizzo con i Componenti {#usage-on-components}

:::warning Not recommended
Using custom directives on components is not recommended. Unexpected behaviour may occur when a component has multiple root nodes.
:::

Quando utilizzate con i componenti, le direttive personalizzate verranno sempre applicate al nodo radice del componente, in modo simile agli [Attributi Trasferibili (Fallthrough)](/guide/components/attrs).

```vue-html
<MyComponent v-demo="test" />
```

```vue-html
<!-- template di MyComponent -->

<div> <!-- la direttiva `v-demo` verrà applicata qui -->
  <span>Contenuto di My component</span>
</div>
```

Nota che i componenti, potenzialmente, possono avere più di un nodo radice. Quando applicata a un componente multi-radice, la direttiva personalizzata verrà ignorata e verrà generato un avviso. A differenza degli attributi, le direttive non possono essere passate a un elemento diverso con `v-bind="$attrs"`. In generale, **non** è consigliato utilizzare direttive personalizzate sui componenti.

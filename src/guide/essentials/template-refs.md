# I Ref del Template {#template-refs}

Mentre il modello di rendering dichiarativo di Vue astrae la maggior parte delle operazioni dirette sul DOM, potrebbero comunque esserci casi in cui abbiamo bisogno di accedere direttamente agli elementi DOM sottostanti. Per raggiungere questo obiettivo, possiamo utilizzare l'attributo speciale `ref`:

```vue-html
<input ref="input">
```

`ref` è un attributo speciale, simile all'attributo `key` discusso nel capitolo `v-for`. Ci permette di ottenere un riferimento diretto a uno specifico elemento DOM o istanza di un componente figlio dopo che è stato montato. Questo potrebbe essere utile, ad esempio, quando si vuole concentrarsi programmaticamente su un input al mount del componente, o inizializzare una libreria di terze parti su un elemento.

## Accesso ai Ref {#accessing-the-refs}

<div class="composition-api">

Per ottenere il riferimento con la Composition API, possiamo usare l'helper [`useTemplateRef()`](/api/composition-api-helpers#usetemplateref) <sup class="vt-badge" data-text="3.5+" />:

```vue
<script setup>
import { useTemplateRef, onMounted } from 'vue'

// the first argument must match the ref value in the template
const input = useTemplateRef('my-input')

onMounted(() => {
  input.value.focus()
})
</script>

<template>
  <input ref="my-input" />
</template>
```

When using TypeScript, Vue's IDE support and `vue-tsc` will automatically infer the type of `input.value` based on what element or component the matching `ref` attribute is used on.

<details>
<summary>Usage before 3.5</summary>

In versions before 3.5 where `useTemplateRef()` was not introduced, we need to declare a ref with a name that matches the template ref attribute's value:

```vue
<script setup>
import { ref, onMounted } from 'vue'

// dichiara un ref per contenere il riferimento all'elemento
// il nome deve corrispondere al valore del ref nel template
const input = ref(null)

onMounted(() => {
  input.value.focus()
})
</script>

<template>
  <input ref="input" />
</template>
```

Se non stai utilizzando `<script setup>`, assicurati di restituire anche il ref da `setup()`:

```js{6}
export default {
  setup() {
    const input = ref(null)
    // ...
    return {
      input
    }
  }
}
```

</details>

</div>
<div class="options-api">

Il ref risultante è esposto su `this.$refs`:

```vue
<script>
export default {
  mounted() {
    this.$refs.input.focus()
  }
}
</script>

<template>
  <input ref="input" />
</template>
```

</div>

Tieni presente che puoi accedere al ref **solo dopo che il componente è montato**. Se provi ad accedere a <span class="options-api">`$refs.input`</span><span class="composition-api">`input`</span> in un'espressione del template, il valore sarà <span class="options-api">`undefined`</span><span class="composition-api">`null`</span> al primo rendering. Questo perché l'elemento non esiste fino al primo rendering!

<div class="composition-api">

Se stai cercando di osservare i cambiamenti di un ref del template, assicurati di considerare il caso in cui il ref ha un valore `null`:

```js
watchEffect(() => {
  if (input.value) {
    input.value.focus()
  } else {
    // l'elemento non è stato ancora montato, o è stato smontato (ad es. da v-if)
  }
})
```

Guarda anche: [Tipizzazione dei Ref del template](/guide/typescript/composition-api#typing-template-refs) <sup class="vt-badge ts" />

</div>

## I Ref all'interno di `v-for` {#refs-inside-v-for}

> Richiede v3.2.25 o superiore

<div class="composition-api">

Quando `ref` viene utilizzato all'interno di `v-for`, il corrispondente ref dovrebbe contenere un valore di tipo Array, che verrà popolato con gli elementi dopo il `mount`:

```vue
<script setup>
import { ref, onMounted } from 'vue'

const list = ref([
  /* ... */
])

const itemRefs = ref([])

onMounted(() => console.log(itemRefs.value))
</script>

<template>
  <ul>
    <li v-for="item in list" ref="itemRefs">
      {{ item }}
    </li>
  </ul>
</template>
```

[Prova nel Playground](https://play.vuejs.org/#eNpFjs1qwzAQhF9l0CU2uDZtb8UOlJ576bXqwaQyCGRJyCsTEHr3rGwnOehnd2e+nSQ+vW/XqMSH6JdL0J6wKIr+LK2evQuEhKCmBs5+u2hJ/SNjCm7GiV0naaW9OLsQjOZrKNrq97XBW4P3v/o51qTmHzUtd8k+e0CrqsZwRpIWGI0KVN0N7TqaqNp59JUuEt2SutKXY5elmimZT9/t2Tk1F+z0ZiTFFdBHs738Mxrry+TCIEWhQ9sttRQl0tEsK6U4HEBKW3LkfDA6o3dst3H77rFM5BtTfm/P)

</div>
<div class="options-api">

Quando `ref` viene utilizzato all'interno di `v-for`, il valore ref risultante sarà un array contenente gli elementi corrispondenti:

```vue
<script>
export default {
  data() {
    return {
      list: [
        /* ... */
      ]
    }
  },
  mounted() {
    console.log(this.$refs.items)
  }
}
</script>

<template>
  <ul>
    <li v-for="item in list" ref="items">
      {{ item }}
    </li>
  </ul>
</template>
```

[Prova nel Playground](https://play.vuejs.org/#eNpFjk0KwjAQha/yCC4Uaou6kyp4DuOi2KkGYhKSiQildzdNa4WQmTc/37xeXJwr35HEUdTh7pXjszT0cdYzWuqaqBm9NEDbcLPeTDngiaM3PwVoFfiI667AvsDhNpWHMQzF+L9sNEztH3C3JlhNpbaPNT9VKFeeulAqplfY5D1p0qurxVQSqel0w5QUUEedY8q0wnvbWX+SYgRAmWxIiuSzm4tBinkc6HvkuSE7TIBKq4lZZWhdLZfE8AWp4l3T)

</div>

Va notato che l'array ref **non** garantisce lo stesso ordine dell'array sorgente.

## I Ref con Funzioni {#function-refs}

Al posto di usare una stringa, l'attributo `ref` può essere legato anche a una funzione, che verrà chiamata ad ogni aggiornamento del componente e ti darà piena flessibilità su dove memorizzare il riferimento dell'elemento. La funzione riceve il riferimento dell'elemento come primo argomento:

```vue-html
<input :ref="(el) => { /* assegna el a una proprietà o a un ref */ }">
```

Nota che stiamo usando un binding `:ref` dinamico in modo da poter passare una funzione invece di una stringa con il nome del ref. Quando l'elemento viene smontato, l'argomento sarà `null`. Puoi, naturalmente, utilizzare un metodo invece di una funzione inline.

## Ref sui Componenti {#ref-on-component}

> Questa sezione assume che tu abbia conoscenza dei [Componenti](/guide/essentials/component-basics). Sentiti libero di saltarla e riprendere più tardi.

`ref` può essere utilizzato anche su un componente figlio. In questo caso, il riferimento sarà quello di un'istanza del componente:

<div class="composition-api">

```vue
<script setup>
import { useTemplateRef, onMounted } from 'vue'
import Child from './Child.vue'

const childRef = useTemplateRef('child')

onMounted(() => {
  // childRef.value will hold an instance of <Child />
})
</script>

<template>
  <Child ref="child" />
</template>
```

<details>
<summary>Usage before 3.5</summary>

```vue
<script setup>
import { ref, onMounted } from 'vue'
import Child from './Child.vue'

const child = ref(null)

onMounted(() => {
  // child.value conterrà un'istanza di <Child />
})
</script>

<template>
  <Child ref="child" />
</template>
```

</details>

</div>
<div class="options-api">

```vue
<script>
import Child from './Child.vue'

export default {
  components: {
    Child
  },
  mounted() {
    // this.$refs.child conterrà un'istanza di <Child />
  }
}
</script>

<template>
  <Child ref="child" />
</template>
```

</div>

<span class="composition-api">Se il componente figlio sta utilizzando l'Options API o non sta utilizzando `<script setup>`, l'</span><span  class="options-api">L'</span> istanza referenziata sarà identica a `this` del componente figlio, il che significa che il componente padre avrà accesso completo a ogni proprietà e metodo del componente figlio. Ciò rende facile creare implementazioni strettamente accoppiati tra il genitore e il figlio, quindi i ref del componente dovrebbero essere utilizzati solo se assolutamente necessario - nella maggior parte dei casi, dovresti cercare di implementare le interazioni genitore/figlio utilizzando prima tramite le interfacce standard di props ed emit.

<div class="composition-api">

Un'eccezione a ciò è che i componenti che utilizzano `<script setup>` sono di **default privati**: un componente padre che fa riferimento a un componente figlio utilizzando `<script setup>` non sarà in grado di accedere a nulla a meno che il componente figlio non scelga di esporre un'interfaccia pubblica utilizzando la macro `defineExpose`:

```vue
<script setup>
import { ref } from 'vue'

const a = 1
const b = ref(2)

// Le macro del compilatore, come defineExpose, non hanno bisogno di essere importate
defineExpose({
  a,
  b
})
</script>
```

Quando un genitore ottiene un'istanza di questo componente tramite i ref del template, l'istanza recuperata avrà la forma `{a: number, b: number}` (i ref vengono estratti automaticamente proprio come nelle normali istanze).

Note that defineExpose must be called before any await operation. Otherwise, properties and methods exposed after the await operation will not be accessible. 

Guarda anche: [Tipizzazione dei Ref del Template dei Componenti](/guide/typescript/composition-api#typing-component-template-refs) <sup class="vt-badge ts" />

</div>
<div class="options-api">

L'opzione `expose` può essere utilizzata per limitare l'accesso a un'istanza figlio:

```js
export default {
  expose: ['publicData', 'publicMethod'],
  data() {
    return {
      publicData: 'foo',
      privateData: 'bar'
    }
  },
  methods: {
    publicMethod() {
      /* ... */
    },
    privateMethod() {
      /* ... */
    }
  }
}
```

Nell'esempio qui sopra, un genitore che fa riferimento a questo componente tramite il ref del template sarà in grado di accedere solo a `publicData` e `publicMethod`.

</div>

## Refs inside `v-for` {#refs-inside-v-for}

> Requires v3.5 or above

<div class="composition-api">

When `ref` is used inside `v-for`, the corresponding ref should contain an Array value, which will be populated with the elements after mount:

```vue
<script setup>
import { ref, useTemplateRef, onMounted } from 'vue'

const list = ref([
  /* ... */
])

const itemRefs = useTemplateRef('items')

onMounted(() => console.log(itemRefs.value))
</script>

<template>
  <ul>
    <li v-for="item in list" ref="items">
      {{ item }}
    </li>
  </ul>
</template>
```

[Try it in the Playground](https://play.vuejs.org/#eNp9UsluwjAQ/ZWRLwQpDepyQoDUIg6t1EWUW91DFAZq6tiWF4oU5d87dtgqVRyyzLw3b+aN3bB7Y4ptQDZkI1dZYTw49MFMuBK10dZDAxZXOQSHC6yNLD3OY6zVsw7K4xJaWFldQ49UelxxVWnlPEhBr3GszT6uc7jJ4fazf4KFx5p0HFH+Kme9CLle4h6bZFkfxhNouAIoJVqfHQSKbSkDFnVpMhEpovC481NNVcr3SaWlZzTovJErCqgydaMIYBRk+tKfFLC9Wmk75iyqg1DJBWfRxT7pONvTAZom2YC23QsMpOg0B0l0NDh2YjnzjpyvxLrYOK1o3ckLZ5WujSBHr8YL2gxnw85lxEop9c9TynkbMD/kqy+svv/Jb9wu5jh7s+jQbpGzI+ZLu0byEuHZ+wvt6Ays9TJIYl8A5+i0DHHGjvYQ1JLGPuOlaR/TpRFqvXCzHR2BO5iKg0Zmm/ic0W2ZXrB+Gve2uEt1dJKs/QXbwePE)

<details>
<summary>Usage before 3.5</summary>

In versions before 3.5 where `useTemplateRef()` was not introduced, we need to declare a ref with a name that matches the template ref attribute's value. The ref should also contain an array value:

```vue
<script setup>
import { ref, onMounted } from 'vue'

const list = ref([
  /* ... */
])

const itemRefs = ref([])

onMounted(() => console.log(itemRefs.value))
</script>

<template>
  <ul>
    <li v-for="item in list" ref="itemRefs">
      {{ item }}
    </li>
  </ul>
</template>
```

</details>

</div>
<div class="options-api">

When `ref` is used inside `v-for`, the resulting ref value will be an array containing the corresponding elements:

```vue
<script>
export default {
  data() {
    return {
      list: [
        /* ... */
      ]
    }
  },
  mounted() {
    console.log(this.$refs.items)
  }
}
</script>

<template>
  <ul>
    <li v-for="item in list" ref="items">
      {{ item }}
    </li>
  </ul>
</template>
```

[Try it in the Playground](https://play.vuejs.org/#eNpFjk0KwjAQha/yCC4Uaou6kyp4DuOi2KkGYhKSiQildzdNa4WQmTc/37xeXJwr35HEUdTh7pXjszT0cdYzWuqaqBm9NEDbcLPeTDngiaM3PwVoFfiI667AvsDhNpWHMQzF+L9sNEztH3C3JlhNpbaPNT9VKFeeulAqplfY5D1p0qurxVQSqel0w5QUUEedY8q0wnvbWX+SYgRAmWxIiuSzm4tBinkc6HvkuSE7TIBKq4lZZWhdLZfE8AWp4l3T)

</div>

It should be noted that the ref array does **not** guarantee the same order as the source array.

## Function Refs {#function-refs}

Instead of a string key, the `ref` attribute can also be bound to a function, which will be called on each component update and gives you full flexibility on where to store the element reference. The function receives the element reference as the first argument:

```vue-html
<input :ref="(el) => { /* assign el to a property or ref */ }">
```

Note we are using a dynamic `:ref` binding so we can pass it a function instead of a ref name string. When the element is unmounted, the argument will be `null`. You can, of course, use a method instead of an inline function.

## Ref on Component {#ref-on-component}

> This section assumes knowledge of [Components](/guide/essentials/component-basics). Feel free to skip it and come back later.

`ref` can also be used on a child component. In this case the reference will be that of a component instance:

<div class="composition-api">

```vue
<script setup>
import { ref, onMounted } from 'vue'
import Child from './Child.vue'

const child = ref(null)

onMounted(() => {
  // child.value will hold an instance of <Child />
})
</script>

<template>
  <Child ref="child" />
</template>
```

</div>
<div class="options-api">

```vue
<script>
import Child from './Child.vue'

export default {
  components: {
    Child
  },
  mounted() {
    // this.$refs.child will hold an instance of <Child />
  }
}
</script>

<template>
  <Child ref="child" />
</template>
```

</div>

<span class="composition-api">If the child component is using Options API or not using `<script setup>`, the</span><span class="options-api">The</span> referenced instance will be identical to the child component's `this`, which means the parent component will have full access to every property and method of the child component. This makes it easy to create tightly coupled implementation details between the parent and the child, so component refs should be only used when absolutely needed - in most cases, you should try to implement parent / child interactions using the standard props and emit interfaces first.

<div class="composition-api">

An exception here is that components using `<script setup>` are **private by default**: a parent component referencing a child component using `<script setup>` won't be able to access anything unless the child component chooses to expose a public interface using the `defineExpose` macro:

```vue
<script setup>
import { ref } from 'vue'

const a = 1
const b = ref(2)

// Compiler macros, such as defineExpose, don't need to be imported
defineExpose({
  a,
  b
})
</script>
```

When a parent gets an instance of this component via template refs, the retrieved instance will be of the shape `{ a: number, b: number }` (refs are automatically unwrapped just like on normal instances).

See also: [Typing Component Template Refs](/guide/typescript/composition-api#typing-component-template-refs) <sup class="vt-badge ts" />

</div>
<div class="options-api">

The `expose` option can be used to limit the access to a child instance:

```js
export default {
  expose: ['publicData', 'publicMethod'],
  data() {
    return {
      publicData: 'foo',
      privateData: 'bar'
    }
  },
  methods: {
    publicMethod() {
      /* ... */
    },
    privateMethod() {
      /* ... */
    }
  }
}
```

In the above example, a parent referencing this component via template ref will only be able to access `publicData` and `publicMethod`.

</div>

## Refs inside `v-for` {#refs-inside-v-for}

> Requires v3.5 or above

<div class="composition-api">

When `ref` is used inside `v-for`, the corresponding ref should contain an Array value, which will be populated with the elements after mount:

```vue
<script setup>
import { ref, useTemplateRef, onMounted } from 'vue'

const list = ref([
  /* ... */
])

const itemRefs = useTemplateRef('items')

onMounted(() => console.log(itemRefs.value))
</script>

<template>
  <ul>
    <li v-for="item in list" ref="items">
      {{ item }}
    </li>
  </ul>
</template>
```

[Try it in the Playground](https://play.vuejs.org/#eNp9UsluwjAQ/ZWRLwQpDepyQoDUIg6t1EWUW91DFAZq6tiWF4oU5d87dtgqVRyyzLw3b+aN3bB7Y4ptQDZkI1dZYTw49MFMuBK10dZDAxZXOQSHC6yNLD3OY6zVsw7K4xJaWFldQ49UelxxVWnlPEhBr3GszT6uc7jJ4fazf4KFx5p0HFH+Kme9CLle4h6bZFkfxhNouAIoJVqfHQSKbSkDFnVpMhEpovC481NNVcr3SaWlZzTovJErCqgydaMIYBRk+tKfFLC9Wmk75iyqg1DJBWfRxT7pONvTAZom2YC23QsMpOg0B0l0NDh2YjnzjpyvxLrYOK1o3ckLZ5WujSBHr8YL2gxnw85lxEop9c9TynkbMD/kqy+svv/Jb9wu5jh7s+jQbpGzI+ZLu0byEuHZ+wvt6Ays9TJIYl8A5+i0DHHGjvYQ1JLGPuOlaR/TpRFqvXCzHR2BO5iKg0Zmm/ic0W2ZXrB+Gve2uEt1dJKs/QXbwePE)

<details>
<summary>Usage before 3.5</summary>

In versions before 3.5 where `useTemplateRef()` was not introduced, we need to declare a ref with a name that matches the template ref attribute's value. The ref should also contain an array value:

```vue
<script setup>
import { ref, onMounted } from 'vue'

const list = ref([
  /* ... */
])

const itemRefs = ref([])

onMounted(() => console.log(itemRefs.value))
</script>

<template>
  <ul>
    <li v-for="item in list" ref="itemRefs">
      {{ item }}
    </li>
  </ul>
</template>
```

</details>

</div>
<div class="options-api">

When `ref` is used inside `v-for`, the resulting ref value will be an array containing the corresponding elements:

```vue
<script>
export default {
  data() {
    return {
      list: [
        /* ... */
      ]
    }
  },
  mounted() {
    console.log(this.$refs.items)
  }
}
</script>

<template>
  <ul>
    <li v-for="item in list" ref="items">
      {{ item }}
    </li>
  </ul>
</template>
```

[Try it in the Playground](https://play.vuejs.org/#eNpFjk0KwjAQha/yCC4Uaou6kyp4DuOi2KkGYhKSiQildzdNa4WQmTc/37xeXJwr35HEUdTh7pXjszT0cdYzWuqaqBm9NEDbcLPeTDngiaM3PwVoFfiI667AvsDhNpWHMQzF+L9sNEztH3C3JlhNpbaPNT9VKFeeulAqplfY5D1p0qurxVQSqel0w5QUUEedY8q0wnvbWX+SYgRAmWxIiuSzm4tBinkc6HvkuSE7TIBKq4lZZWhdLZfE8AWp4l3T)

</div>

It should be noted that the ref array does **not** guarantee the same order as the source array.

## Function Refs {#function-refs}

Instead of a string key, the `ref` attribute can also be bound to a function, which will be called on each component update and gives you full flexibility on where to store the element reference. The function receives the element reference as the first argument:

```vue-html
<input :ref="(el) => { /* assign el to a property or ref */ }">
```

Note we are using a dynamic `:ref` binding so we can pass it a function instead of a ref name string. When the element is unmounted, the argument will be `null`. You can, of course, use a method instead of an inline function.

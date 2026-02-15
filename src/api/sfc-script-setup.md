# \<script setup> {#script-setup}

`<script setup>` è uno dei modi per usare la Composition API dentro ai Single-File Components (SFC).  È la sintassi consigliata se stai utilizzando sia i componenti SFC che la Composition API. Fornisce diversi vantaggi rispetto alla normale sintassi `<script>`:

- Codice più succinto con meno boilerplate
- Possibilità di dichiarare props ed eventi emessi usando TypeScript puro
- Migliore performance a runtime (il template viene compilato in una funzione di rendering nello stesso scope, senza un proxy intermedio)
- Migliore performance di inferenza del tipo nell'IDE (meno lavoro per il language server per estrarre i tipi dal codice)

## Sintassi base {#basic-syntax}

Per utilizzare a questa sintassi, aggiungi l'attributo `setup` al blocco `<script>`:

```vue
<script setup>
console.log('Ciao, script setup!')
</script>
```

Il codice al suo interno viene compilato come contenuto della funzione `setup()` del componente. Ciò significa che, a differenza di un normale `<script>`, che si esegue solo una volta quando il componente viene importato per la prima volta, il codice all'interno di `<script setup>` verrà eseguito **ogni volta che viene creato un'istanza del componente**.

### I top-level bindings sono esposti al template {#top-level-bindings-are-exposed-to-template}

Quando si utilizza `<script setup>`, tutti i top-level bindings (incluse variabili, dichiarazioni di funzioni e importazioni) definite all'interno di `<script setup>` sono direttamente utilizzabili nel template:

```vue
<script setup>
// variabile
const msg = 'Ciao!'

// funzione
function log() {
  console.log(msg)
}
</script>

<template>
  <button @click="log">{{ msg }}</button>
</template>
```

Le importazioni sono esposte allo stesso modo. Ciò significa che è possibile utilizzare direttamente una funzione di utility importata nelle espressioni del template senza doverla esporre tramite l'opzione `methods`:

```vue
<script setup>
import { capitalize } from './helpers'
</script>

<template>
  <div>{{ capitalize('ciao') }}</div>
</template>
```

## Reattività {#reactivity}

Lo stato reattivo deve essere creato esplicitamente utilizzando le [API di reattività](./reactivity-core). Similmente ai valori restituiti da una funzione `setup()`, i `ref` vengono automaticamente estratti quando vengono usati nei template:

```vue
<script setup>
import { ref } from 'vue'

const count = ref(0)
</script>

<template>
  <button @click="count++">{{ count }}</button>
</template>
```

## Uso di componenti {#using-components}

I valori nello scope di `<script setup>` possono essere utilizzati direttamente come nomi di tag per i componenti personalizzati:

```vue
<script setup>
import MyComponent from './MyComponent.vue'
</script>

<template>
  <MyComponent />
</template>
```

Pensa a `MyComponent` come se fosse referenziato come una variabile. Se hai utilizzato JSX, il modello mentale è simile qui. L'equivalente in kebab-case `<my-component>` funziona anche nel template, ma è fortemente consigliato utilizzare PascalCase per la coerenza. Ciò aiuta anche a distinguerlo dagli elementi personalizzati nativi.

### Componenti dinamici {#dynamic-components}

Poiché i componenti sono utilizzati come variabili invece che registrati con chiavi di stringa, dovremmo utilizzare il binding dinamico `:is` quando si utilizzano componenti dinamici all'interno di `<script setup>`:

```vue
<script setup>
import Foo from './Foo.vue'
import Bar from './Bar.vue'
</script>

<template>
  <component :is="Foo" />
  <component :is="someCondition ? Foo : Bar" />
</template>
```

Nota come i componenti possono essere utilizzati come variabili in un'espressione ternaria.

### Componenti ricorsivi {#recursive-components}

Un SFC può fare riferimento implicito a se stesso tramite il proprio nome file. Ad esempio, un file chiamato `FooBar.vue` può fare riferimento a se stesso come `<FooBar/>` nel suo template.

Nota che ciò ha una priorità inferiore rispetto ai componenti importati. Se hai un import con nome che entra in conflitto con il nome inferito del componente, puoi creare un alias per l'import:

```js
import { FooBar as FooBarChild } from './components'
```

### Componenti col namespace {#namespaced-components}

Puoi utilizzare tag dei componenti con punti come `<Foo.Bar>` per fare riferimento a componenti annidati sotto proprietà di oggetti. Questo è utile quando importi più componenti da un singolo file:

```vue
<script setup>
import * as Form from './form-components'
</script>

<template>
  <Form.Input>
    <Form.Label>label</Form.Label>
  </Form.Input>
</template>
```

## Uso di direttive personalizzate {#using-custom-directives}

L'uso di direttive personalizzate registrate globalmente funziona normalmente. Le direttive personalizzate locali non devono essere registrate esplicitamente con `<script setup>`, ma devono seguire lo schema di denominazione `vNameOfDirective`:

```vue
<script setup>
const vMyDirective = {
  beforeMount: (el) => {
    // fai qualcosa con l'elemento
  }
}
</script>
<template>
  <h1 v-my-directive>Questo è un titolo</h1>
</template>
```

Se stai importando una direttiva da altrove, puoi rinominarla per adattarla allo schema di denominazione richiesto:

```vue
<script setup>
import { myDirective as vMyDirective } from './MyDirective.js'
</script>
```

## defineProps() & defineEmits() {#defineprops-defineemits}

Per dichiarare opzioni come `props` and `emits` con il supporto completo per l'inferenza dei tipi, possiamo utilizzare le API `defineProps` e `defineEmits`, che sono disponibili automaticamente all'interno di `<script setup>`:

```vue
<script setup>
const props = defineProps({
  foo: String
})

const emit = defineEmits(['change', 'delete'])
// codice di setup
</script>
```

- `defineProps` e `defineEmits` sono  **macro del compilatore** utilizzabili solo all'interno di `<script setup>`. Non è necessario importarli e vengono eliminati durante l'elaborazione di `<script setup>`.

- `defineProps` accetta lo stesso valore dell'opzione `props`, mentre `defineEmits` accetta lo stesso valore dell'opzione `emits`.

- `defineProps` e `defineEmits` forniscono un'adeguata inferenza dei tipi in base alle opzioni passate.

- Le opzioni passate a `defineProps` e `defineEmits` saranno rimosse dal setup nello scope del modulo. Quindi, le opzioni non possono fare riferimento a variabili locali dichiarate nello scope del setup. Farlo causerà un errore di compilazione. Tuttavia, _possono_ fare riferimento a binding importati poiché anch'essi si trovano nello scope del modulo.

### Dichiarazioni di props/emit (solo) per il type<sup class="vt-badge ts" /> {#type-only-props-emit-declarations}

Props ed emits possono essere dichiarati usando una sintassi puramente di tipo passando un argomento di tipo letterale a `defineProps` o `defineEmits`:

```ts
const props = defineProps<{
  foo: string
  bar?: number
}>()

const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
}>()

// 3.3+: sintassi alternativa, ridotta
const emit = defineEmits<{
  change: [id: number] // sintassi di tuple nominata
  update: [value: string]
}>()
```

- `defineProps` o `defineEmits` possono utilizzare solo la dichiarazione a runtime O la dichiarazione di tipo. Utilizzare entrambi contemporaneamente causerà un errore di compilazione.

- Quando si utilizza la dichiarazione di tipo, la dichiarazione a runtime equivalente viene generata automaticamente dall'analisi statica per eliminare la necessità di una doppia dichiarazione e garantire comunque un corretto comportamento a runtime.

  - In modalità di sviluppo, il compilatore cercherà di dedurre la corrispondente validazione a runtime dai tipi. Ad esempio, qui `foo: String` è inferito dal tipo `foo: string`. Se il tipo è un riferimento a un tipo importato, il risultato dedotto sarà `foo: null` (equivalente al tipo `any`) poiché il compilatore non ha informazioni sui file esterni.

  - In produzione, il compilatore genererà la dichiarazione in formato array per ridurre le dimensioni del bundle (qui le props saranno compilate in  `['foo', 'bar']`)

- Nella versione 3.2 e precedenti, il parametro del tipo generico per `defineProps()` era limitato a un tipo letterale o a un riferimento a un'interfaccia locale.

  Questa limitazione è stata risolta nella versione 3.3. La versione più recente di Vue supporta il riferimento a tipi importati e a un insieme limitato di tipi complessi nella posizione del parametro di tipo. Tuttavia, poiché la conversione da tipo a runtime è ancora basata sull'AST, alcuni tipi complessi che richiedono un'effettiva analisi del tipo, ad esempio i tipi condizionali, non sono supportati. È possibile utilizzare tipi condizionali per il tipo di una singola prop, ma non per l'intero oggetto delle props.

### Reactive Props Destructure <sup class="vt-badge" data-text="3.5+" /> {#reactive-props-destructure}

In Vue 3.5 and above, variables destructured from the return value of `defineProps` are reactive. Vue's compiler automatically prepends `props.` when code in the same `<script setup>` block accesses variables destructured from `defineProps`:

```ts
const { foo } = defineProps(['foo'])

watchEffect(() => {
  // runs only once before 3.5
  // re-runs when the "foo" prop changes in 3.5+
  console.log(foo)
})
```

The above is compiled to the following equivalent:

```js {5}
const props = defineProps(['foo'])

watchEffect(() => {
  // `foo` transformed to `props.foo` by the compiler
  console.log(props.foo)
})
```

In addition, you can use JavaScript's native default value syntax to declare default values for the props. This is particularly useful when using the type-based props declaration:

```ts
interface Props {
  msg?: string
  labels?: string[]
}

const { msg = 'hello', labels = ['one', 'two'] } = defineProps<Props>()
```

### Valori predefiniti delle props quando si utilizza la dichiarazione di tipo <sup class="vt-badge ts" /> {#default-props-values-when-using-type-declaration}

In 3.5 and above, default values can be naturally declared when using Reactive Props Destructure. But in 3.4 and below, Reactive Props Destructure is not enabled by default. In order to declare props default values with type-based declaration, the `withDefaults` compiler macro is needed:

```ts
interface Props {
  msg?: string
  labels?: string[]
}

const props = withDefaults(defineProps<Props>(), {
  msg: 'Ciao',
  labels: () => ['one', 'two']
})
```

Questo verrà compilato a runtime nelle equivalenti opzioni `default` per le props. Inoltre, l'helper `withDefaults` fornisce controlli di tipo per i valori predefiniti e garantisce che il tipo delle `props` restituito non abbia i flag opzionali per le proprietà che hanno valori predefiniti dichiarati.

:::info
Note that default values for mutable reference types (like arrays or objects) should be wrapped in functions when using `withDefaults` to avoid accidental modification and external side effects. This ensures each component instance gets its own copy of the default value. This is **not** necessary when using default values with destructure.
:::

## defineModel() {#definemodel}

- Only available in 3.4+

This macro can be used to declare a two-way binding prop that can be consumed via `v-model` from the parent component. Example usage is also discussed in the [Component `v-model`](/guide/components/v-model) guide.

Under the hood, this macro declares a model prop and a corresponding value update event. If the first argument is a literal string, it will be used as the prop name; Otherwise the prop name will default to `"modelValue"`. In both cases, you can also pass an additional object which can include the prop's options and the model ref's value transform options.

```js
// declares "modelValue" prop, consumed by parent via v-model
const model = defineModel()
// OR: declares "modelValue" prop with options
const model = defineModel({ type: String })

// emits "update:modelValue" when mutated
model.value = 'hello'

// declares "count" prop, consumed by parent via v-model:count
const count = defineModel('count')
// OR: declares "count" prop with options
const count = defineModel('count', { type: Number, default: 0 })

function inc() {
  // emits "update:count" when mutated
  count.value++
}
```

:::warning
If you have a `default` value for `defineModel` prop and you don't provide any value for this prop from the parent component, it can cause a de-synchronization between parent and child components. In the example below, the parent's `myRef` is undefined, but the child's `model` is 1:

```vue [Child.vue]
<script setup>
const model = defineModel({ default: 1 })
</script>
```

```vue [Parent.vue]
<script setup>
const myRef = ref()
</script>

<template>
  <Child v-model="myRef"></Child>
</template>
```

:::

### Modifiers and Transformers {#modifiers-and-transformers}

To access modifiers used with the `v-model` directive, we can destructure the return value of `defineModel()` like this:

```js
const [modelValue, modelModifiers] = defineModel()

// corresponds to v-model.trim
if (modelModifiers.trim) {
  // ...
}
```

When a modifier is present, we likely need to transform the value when reading or syncing it back to the parent. We can achieve this by using the `get` and `set` transformer options:

```js
const [modelValue, modelModifiers] = defineModel({
  // get() omitted as it is not needed here
  set(value) {
    // if the .trim modifier is used, return trimmed value
    if (modelModifiers.trim) {
      return value.trim()
    }
    // otherwise, return the value as-is
    return value
  }
})
```

### Usage with TypeScript <sup class="vt-badge ts" /> {#usage-with-typescript}

Like `defineProps` and `defineEmits`, `defineModel` can also receive type arguments to specify the types of the model value and the modifiers:

```ts
const modelValue = defineModel<string>()
//    ^? Ref<string | undefined>

// default model with options, required removes possible undefined values
const modelValue = defineModel<string>({ required: true })
//    ^? Ref<string>

const [modelValue, modifiers] = defineModel<string, 'trim' | 'uppercase'>()
//                 ^? Record<'trim' | 'uppercase', true | undefined>
```

## defineExpose() {#defineexpose}

I componenti che utilizzano `<script setup>` sono **chiusi per impostazione predefinita** - ovvero l'istanza pubblica del componente, ottenuta tramite riferimenti nel template o "catene" `$parent`, **non** espone alcun binding dichiarato all'interno di `<script setup>`.

Per esporre esplicitamente proprietà in un componente `<script setup>`, usa la macro del compilatore `defineExpose`:

```vue
<script setup>
import { ref } from 'vue'

const a = 1
const b = ref(2)

defineExpose({
  a,
  b
})
</script>
```

Quando un componente genitore ottiene un'istanza di questo componente tramite riferimenti nel template, l'istanza ottenuta avrà la forma `{ a: number, b: number }` (i riferimenti vengono automaticamente estratti come nelle istanze normali).

## defineOptions() {#defineoptions}

- Only supported in 3.3+

Questa macro può essere utilizzata per dichiarare le opzioni del componente direttamente all'interno di `<script setup>` senza dover utilizzare un blocco `<script>` separato:

```vue
<script setup>
defineOptions({
  inheritAttrs: false,
  customOptions: {
    /* ... */
  }
})
</script>
```

- Questa è una macro. Le opzioni saranno collocate nello scope del modulo e non possono accedere alle variabili locali in `<script setup>` che non sono costanti letterali.

## defineSlots()<sup class="vt-badge ts"/> {#defineslots}

- Only supported in 3.3+

Questa macro può essere utilizzata per fornire suggerimenti di tipo agli IDE per il controllo dei nomi degli slot e dei tipi delle props.

`defineSlots()` accetta solo un parametro di tipo e nessun argomento a runtime. Il parametro di tipo dovrebbe essere un tipo letterale in cui la chiave della proprietà è il nome dello slot e il tipo del valore è la funzione dello slot. Il primo argomento della funzione è la props che lo slot si aspetta di ricevere, e il suo tipo sarà usato per le props dello slot nel template. Il tipo di ritorno viene attualmente ignorato e può essere `any`, ma potremmo sfruttarlo per il controllo del contenuto dello slot in futuro.

Restituisce anche l'oggetto `slots`, che è equivalente all'oggetto `slots` esposto nel contesto di setup o restituito da `useSlots()`.

```vue
<script setup lang="ts">
const slots = defineSlots<{
  default(props: { msg: string }): any
}>()
</script>
```

## `useSlots()` & `useAttrs()` {#useslots-useattrs}

L'utilizzo di `slots` e `attrs` all'interno di `<script setup>` dovrebbe essere relativamente raro, poiché è possibile accedervi direttamente come `$slots` e `$attrs` nel template. Nel raro caso in cui ne abbiate bisogno, utilizzate gli helper `useSlots` e `useAttrs` rispettivamente:

```vue
<script setup>
import { useSlots, useAttrs } from 'vue'

const slots = useSlots()
const attrs = useAttrs()
</script>
```

`useSlots` e `useAttrs` sono effettive funzioni runtime che restituiscono l'equivalente di `setupContext.slots` e `setupContext.attrs`. Possono essere utilizzate anche nelle normali funzioni della composition API.

## Utilizzo insieme al classico `<script>` {#usage-alongside-normal-script}

`<script setup>` può essere utilizzato insieme a `<script>` classico. Potrebbe essere necessario uno `<script>` classico nei casi:

- Dichiarare opzioni che non possono essere espresse in `<script setup>`, ad esempio `inheritAttrs` o opzioni personalizzate abilitate tramite plugin (possono essere sostituite da [`defineOptions`](/api/sfc-script-setup#defineoptions) in 3.3+).
- Dichiarare export con nome.
- Eseguire effetti collaterali o creare oggetti che devono essere eseguiti solo una volta.

```vue
<script>
// <script> classico, eseguito nello scope del modulo (solo una volta)
runSideEffectOnce()

// dichiarare opzioni aggiuntive
export default {
  inheritAttrs: false,
  customOptions: {}
}
</script>

<script setup>
// eseguito nello scope di setup() (per ogni istanza)
</script>
```

Il supporto per combinare di `<script setup>` e `<script>` nello stesso componente è limitato agli scenari descritti sopra. Nello specifico:

- **NON** utilizzare una sezione `<script>` separata per le opzioni che possono già essere definite usando `<script setup>`, come `props` e `emits`.
- Le variabili create all'interno di `<script setup>` non vengono aggiunte come proprietà all'istanza del componente, rendendole non accessibili dalla Options API. Si sconsiglia vivamente di mixare le API in questo modo.

Se vi trovate in uno degli scenari non supportati, dovreste considerare di passare a una funzione [`setup()`](/api/composition-api-setup)  esplicita, invece di utilizzare `<script setup>`.

## `await` top-level {#top-level-await}

È possibile utilizzare `await` al livello superiore all'interno di `<script setup>`. Il codice risultante sarà compilato come `async setup()`:

```vue
<script setup>
const post = await fetch(`/api/post/1`).then((r) => r.json())
</script>
```

Inoltre, l'espressione attesa verrà compilata automaticamente in un formato che conserva il contesto dell'istanza del componente corrente dopo l'`await`.

:::warning Note
`async setup()` deve essere utilizzato in combinazione con [`Suspense`](/guide/built-ins/suspense.html), che è attualmente ancora una funzionalità sperimentale. Prevediamo di finalizzarla e documentarla in una futura versione, ma se siete curiosi ora, potete fare riferimento ai suoi [tests](https://github.com/vuejs/core/blob/main/packages/runtime-core/__tests__/components/Suspense.spec.ts) per vedere come funziona.
:::

## Import Statements {#imports-statements}

Import statements in vue follow [ECMAScript module specification](https://nodejs.org/api/esm.html).
In addition, you can use aliases defined in your build tool configuration:

```vue
<script setup>
import { ref } from 'vue'
import { componentA } from './Components'
import { componentB } from '@/Components'
import { componentC } from '~/Components'
</script>
```

## Generics <sup class="vt-badge ts" /> {#generics}

I parametri di tipo generico possono essere dichiarati utilizzando l'attributo `generic` sul tag `<script>`:

```vue
<script setup lang="ts" generic="T">
defineProps<{
  items: T[]
  selected: T
}>()
</script>
```

Il valore di `generic` funziona esattamente allo stesso modo della lista dei parametri tra `<...>` in TypeScript. Ad esempio, è possibile utilizzare parametri multipli, vincoli `extends`, tipi predefiniti e fare riferimento a tipi importati:

```vue
<script
  setup
  lang="ts"
  generic="T extends string | number, U extends Item"
>
import type { Item } from './types'
defineProps<{
  id: T
  list: U[]
}>()
</script>
```

You can use `@vue-generic` the directive to pass in explicit types, for when the type cannot be inferred:

```vue
<template>
  <!-- @vue-generic {import('@/api').Actor} -->
  <ApiSelect v-model="peopleIds" endpoint="/api/actors" id-prop="actorId" />

  <!-- @vue-generic {import('@/api').Genre} -->
  <ApiSelect v-model="genreIds" endpoint="/api/genres" id-prop="genreId" />
</template>
```

In order to use a reference to a generic component in a `ref` you need to use the [`vue-component-type-helpers`](https://www.npmjs.com/package/vue-component-type-helpers) library as `InstanceType` won't work.

```vue
<script
  setup
  lang="ts"
>
import componentWithoutGenerics from '../component-without-generics.vue';
import genericComponent from '../generic-component.vue';

import type { ComponentExposed } from 'vue-component-type-helpers';

// Works for a component without generics
ref<InstanceType<typeof componentWithoutGenerics>>();

ref<ComponentExposed<typeof genericComponent>>();
```

```vue
<template>
  <!-- @vue-generic {import('@/api').Actor} -->
  <ApiSelect v-model="peopleIds" endpoint="/api/actors" id-prop="actorId" />

  <!-- @vue-generic {import('@/api').Genre} -->
  <ApiSelect v-model="genreIds" endpoint="/api/genres" id-prop="genreId" />
</template>
```

In order to use a reference to a generic component in a `ref` you need to use the [`vue-component-type-helpers`](https://www.npmjs.com/package/vue-component-type-helpers) library as `InstanceType` won't work.

```vue
<script
  setup
  lang="ts"
>
import componentWithoutGenerics from '../component-without-generics.vue';
import genericComponent from '../generic-component.vue';

import type { ComponentExposed } from 'vue-component-type-helpers';

// Works for a component without generics
ref<InstanceType<typeof componentWithoutGenerics>>();

ref<ComponentExposed<typeof genericComponent>>();
```

## Restrizioni {#restrictions}

- A causa della differenza nelle modalità di esecuzione dei moduli, il codice all'interno di `<script setup>` si basa sul contesto di un SFC. Quando spostato in file esterni `.js` o `.ts`, può causare confusione sia per gli sviluppatori che per gli strumenti. Pertanto,  **`<script setup>`** non può essere utilizzato con l'attributo `src`.
- `<script setup>` does not support In-DOM Root Component Template.([Related Discussion](https://github.com/vuejs/core/issues/8391))

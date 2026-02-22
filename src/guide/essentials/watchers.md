# I Watcher {#watchers}

## Esempio Base {#basic-example}

Le computed properties ci permettono di calcolare in maniera dichiarativa dei valori derivati. Tuttavia, ci sono casi in cui abbiamo bisogno di eseguire "side effects" in risposta ai cambiamenti di stato - ad esempio: fare modifiche al DOM o cambiare un altro pezzo di stato in base al risultato di un'operazione asincrona.

<div class="options-api">

Con l'Options API possiamo usare l'[opzione `watch`](/api/options-state#watch) per attivare una funzione ogni volta che una proprietà reattiva cambia:

```js
export default {
  data() {
    return {
      question: '',
      answer: 'Le domande contengono di solito un punto interrogativo. ;-)',
      loading: false
    }
  },
  watch: {
    // ogni volta che question cambia, questa funzione verrà eseguita
    question(newQuestion, oldQuestion) {
      if (newQuestion.includes('?')) {
        this.getAnswer()
      }
    }
  },
  methods: {
    async getAnswer() {
      this.loading = true
      this.answer = 'Sto pensando...'
      try {
        const res = await fetch('https://yesno.wtf/api')
        this.answer = (await res.json()).answer
      } catch (error) {
        this.answer = 'Errore! Impossibile raggiungere l\'API. ' + error
      } finally {
        this.loading = false
      }
    }
  }
}
```

```vue-html
<p>
  Fai una domanda a risposta sì/no:
  <input v-model="question" :disabled="loading" />
</p>
<p>{{ answer }}</p>
```

[Prova nel Playground](https://play.vuejs.org/#eNp9VE1v2zAM/SucLnaw1D70lqUbsiKH7rB1W4++aDYdq5ElTx9xgiD/fbT8lXZFAQO2+Mgn8pH0mW2aJjl4ZCu2trkRjfucKTw22jgosOReOjhnCqDgjseL/hvAoPNGjSeAvx6tE1qtIIqWo5Er26Ih088BteCt51KeINfKcaGAT5FQc7NP4NPNYiaQmhdC7VZQcmlxMF+61yUcWu7yajVmkabQVqjwgGZmzSuudmiX4CphofQqD+ZWSAnGqz5y9I4VtmOuS9CyGA9T3QCihGu3RKhc+gJtHH2JFld+EG5Mdug2QYZ4MSKhgBd11OgqXdipEm5PKoer0Jk2kA66wB044/EF1GtOSPRUCbUnryRJosnFnK4zpC5YR7205M9bLhyUSIrGUeVcY1dpekKrdNK6MuWNiKYKXt8V98FElDxbknGxGLCpZMi7VkGMxmjzv0pz1tvO4QPcay8LULoj5RToKoTN40MCEXyEQDJTl0KFmXpNOqsUxudN+TNFzzqdJp8ODutGcod0Alg34QWwsXsaVtIjVXqe9h5bC9V4B4ebWhco7zI24hmDVSEs/yOxIPOQEFnTnjzt2emS83nYFrhcevM6nRJhS+Ys9aoUu6Av7WqoNWO5rhsh0fxownplbBqhjJEmuv0WbN2UDNtDMRXm+zfsz/bY2TL2SH1Ec8CMTZjjhqaxh7e/v+ORvieQqvaSvN8Bf6HV0veSdG5fvSoo7Su/kO1D3f13SKInuz06VHYsahzzfl0yRj+s+3dKn9O9TW7HPrPLP624lFU=)

L'opzione `watch` supporta anche un percorso di chiavi separate da punti:

```js
export default {
  watch: {
    // Nota: solo percorsi semplici. Le espressioni non sono supportate.
    'some.nested.key'(newValue) {
      // ...
    }
  }
}
```

</div>

<div class="composition-api">

Con la Composition API possiamo utilizzare la [funzione `watch`](/api/reactivity-core#watch) per innescare una callback ogni volta che cambia uno stato reattivo:

```vue
<script setup>
import { ref, watch } from 'vue'

const question = ref('')
const answer = ref('Le domande contengono di solito un punto interrogativo. ;-)')
const loading = ref(false)

// watch funziona direttamente su un ref
watch(question, async (newQuestion, oldQuestion) => {
  if (newQuestion.includes('?')) {
    loading.value = true
    answer.value = 'Sto pensando...'
    try {
      const res = await fetch('https://yesno.wtf/api')
      answer.value = (await res.json()).answer
    } catch (error) {
      answer.value = 'Errore! Impossibile raggiungere l\'API. ' + error
    } finally {
      loading.value = false
    }
  }
})
</script>

<template>
  <p>
    Fai una domanda a risposta sì/no:
    <input v-model="question" :disabled="loading" />
  </p>
  <p>{{ answer }}</p>
</template>
```

[Prova nel Playground](https://play.vuejs.org/#eNp9U8Fy0zAQ/ZVFF9tDah96C2mZ0umhHKBAj7oIe52oUSQjyXEyGf87KytyoDC9JPa+p+e3b1cndtd15b5HtmQrV1vZeXDo++6Wa7nrjPVwAovtAgbh6w2M0Fqzg4xOZFxzXRvtPPzq0XlpNNwEbp5lRUKEdgPaVP925jnoXS+UOgKxvJAaxEVjJ+y2hA9XxUVFGdFIvT7LtEI5JIzrqjrbGozdOmikxdqTKqmIQOV6gvOkvQDhjrqGXOOQvCzAqCa9FHBzCyeuAWT7F6uUulZ9gy7PPmZFETmQjJV7oXoke972GJHY+Axkzxupt4FalhRcYHh7TDIQcqA+LTriikFIDy0G59nG+84tq+qITpty8G0lOhmSiedefSaPZ0mnfHFG50VRRkbkj1BPceVorbFzF/+6fQj4O7g3vWpAm6Ao6JzfINw9PZaQwXuYNJJuK/U0z1nxdTLT0M7s8Ec/I3WxquLS0brRi8ddp4RHegNYhR0M/Du3pXFSAJU285osI7aSuus97K92pkF1w1nCOYNlI534qbCh8tkOVasoXkV1+sjplLZ0HGN5Vc1G2IJ5R8Np5XpKlK7J1CJntdl1UqH92k0bzdkyNc8ZRWGGz1MtbMQi1esN1tv/1F/cIdQ4e6LJod0jZzPmhV2jj/DDjy94oOcZpK57Rew3wO/ojOpjJIH2qdcN2f6DN7l9nC47RfTsHg4etUtNpZUeJz5ndPPv32j9Yve6vE6DZuNvu1R2Tg==)

### Tipi di Valori Osservati {#watch-source-types}

Il primo argomento di `watch` può avere diversi tipi di "sorgenti" reattive: può essere un ref (inclusi i ref calcolati), un oggetto reattivo, una [getter function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get#description) o un array di valori multipli:

```js
const x = ref(0)
const y = ref(0)

// ref singola
watch(x, (newX) => {
  console.log(`x è ${newX}`)
})

// getter
watch(
  () => x.value + y.value,
  (sum) => {
    console.log(`la somma di x + y è: ${sum}`)
  }
)

// array di valori multipli
watch([x, () => y.value], ([newX, newY]) => {
  console.log(`x è ${newX} e y è ${newY}`)
})
```

Nota che non puoi osservare una proprietà di un oggetto reattivo in questo modo:

```js
const obj = reactive({ count: 0 })

// questo non funzionerà perché stiamo passando un numero a watch()
watch(obj.count, (count) => {
  console.log(`Count è: ${count}`)
})
```

Invece, utilizza un getter:

```js
// instead, use a getter:
watch(
  () => obj.count,
  (count) => {
    console.log(`Count è: ${count}`)
  }
)
```

</div>

## Watcher Avanzati {#deep-watchers}

<div class="options-api">

Di default, un `watch` è di tipo "shallow" (superficiale): la callback verrà attivata solo quando alla proprietà osservata viene assegnato un nuovo valore - non verrà attivata in caso di modifiche alle proprietà annidate. Se vuoi che la callback venga eseguita per tutte le modifiche effettuate alle proprietà annidate, devi utilizzare un watcher di tipo "deep" (avanzato):

```js
export default {
  watch: {
    someObject: {
      handler(newValue, oldValue) {
        // Nota: qui `newValue` sarà uguale a `oldValue`
        // osu mutazioni annidate fintanto che l'oggetto stesso
        // non è stato sostituito.
      },
      deep: true
    }
  }
}
```

</div>

<div class="composition-api">

Quando chiami `watch()` direttamente su un oggetto reattivo, verrà creato implicitamente un watcher avanzato - la callback verrà attivata su tutte le mutazioni annidate:

```js
const obj = reactive({ count: 0 })

watch(obj, (newValue, oldValue) => {
  // si attiva con mutazioni di proprietà annidate
  // Nota: qui newValue sarà uguale a oldValue
  // perché entrambi puntano allo stesso oggetto!
})

obj.count++
```

Questo dovrebbe essere differenziato da un getter che restituisce un oggetto reattivo - nel secondo caso, la callback verrà attivata solo se il getter restituisce un oggetto diverso:

```js
watch(
  () => state.someObject,
  () => {
    // si attiva solo quando state.someObject viene sostituito
  }
)
```

Puoi forzare, tuttavia, il secondo caso in un watcher avanzato utilizzando esplicitamente l'opzione `deep`:

```js
watch(
  () => state.someObject,
  (newValue, oldValue) => {
    // Nota: qui newValue sarà uguale a oldValue
    // *a meno che* state.someObject non sia stato sostituito
  },
  { deep: true }
)
```

</div>

In Vue 3.5+, the `deep` option can also be a number indicating the max traversal depth - i.e. how many levels should Vue traverse an object's nested properties.

:::warning Usa con Cautela
Il watcher avanzato richiede il traversing di tutte le proprietà annidate nell'oggetto osservato e può essere costoso se utilizzato su grandi strutture dati. Usalo solo quando necessario e tieni conto delle implicazioni sulle prestazioni.
:::

## Watcher Immediati {#eager-watchers}

Di default `watch` è "pigro": la callback non verrà chiamata finché la fonte osservata non cambierà. Ma in alcuni casi potremmo volere che la stessa logica di callback venga eseguita immediatamente - ad esempio, potremmo voler recuperare alcuni dati iniziali e poi ri-recuperare i dati ogni volta che ci siano modifiche dello stato.

<div class="options-api">

Possiamo forzare l'esecuzione immediata della callback di un watcher dichiarandolo tramite un oggetto con una funzione `handler` e l'opzione `immediate: true`:

```js
export default {
  // ...
  watch: {
    question: {
      handler(newQuestion) {
        // questo verrà eseguito immediatamente alla creazione del componente.
      },
      // forza l'esecuzione immediata della callback
      immediate: true
    }
  }
  // ...
}
```

L'esecuzione iniziale della funzione handler avverrà appena prima dell'hook `created`. Vue avrà già elaborato le opzioni `data`, `computed` e `methods`, quindi queste proprietà saranno disponibili alla prima invocazione.

</div>

<div class="composition-api">

Possiamo forzare l'esecuzione immediata della callback di un watcher passando l'opzione `immediate: true`:

```js
watch(
  source,
  (newValue, oldValue) => {
    // eseguito immediatamente, poi di nuovo quando `source` cambia
  },
  { immediate: true }
)
```

</div>

## Once Watchers {#once-watchers}

- Only supported in 3.4+

Watcher's callback will execute whenever the watched source changes. If you want the callback to trigger only once when the source changes, use the `once: true` option.

<div class="options-api">

```js
export default {
  watch: {
    source: {
      handler(newValue, oldValue) {
        // when `source` changes, triggers only once
      },
      once: true
    }
  }
}
```

</div>

<div class="composition-api">

```js
watch(
  source,
  (newValue, oldValue) => {
    // when `source` changes, triggers only once
  },
  { once: true }
)
```

</div>

<div class="composition-api">

## `watchEffect()` \*\* {#watcheffect}

È comune che la callback del watcher utilizzi esattamente lo stesso stato reattivo del valore osservato. Ad esempio, considera il seguente codice, che utilizza un watcher per caricare una risorsa remota ogni volta che la ref `todoId` cambia:

```js
const todoId = ref(1)
const data = ref(null)

watch(
  todoId,
  async () => {
    const response = await fetch(
      `https://jsonplaceholder.typicode.com/todos/${todoId.value}`
    )
    data.value = await response.json()
  },
  { immediate: true }
)
```

In particolare notiamo come il watcher utilizzi `todoId` due volte, una volta come valore osservato e poi di nuovo all'interno della callback.

Questo può essere semplificato con [`watchEffect()`](/api/reactivity-core#watcheffect). `watchEffect()` ci permette di tenere traccia automaticamente delle dipendenze reattive della callback. Il watcher sopra può essere riscritto come:

```js
watchEffect(async () => {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/todos/${todoId.value}`
  )
  data.value = await response.json()
})
```

Qui, la callback verrà eseguita immediatamente, senza bisogno di specificare `immediate: true`. Durante la sua esecuzione, terrà automaticamente traccia di `todoId.value` come dipendenza (come nelle computed properties). Ogni volta che `todoId.value` cambia, la callback verrà eseguita di nuovo. Con `watchEffect()` non c'è più bisogno di passare `todoId` esplicitamente come valore osservato.

Puoi vedere [questo esempio](/examples/#fetching-data) di `watchEffect()` e del recupero di dati reattivi in azione.

Per questo tipo di esempi, con una sola dipendenza, il vantaggio di `watchEffect()` è relativamente piccolo. Ma per i watcher che hanno molteplici dipendenze, l'utilizzo di `watchEffect()` elimina la responsabilità di dover mantenere manualmente l'elenco delle dipendenze. Inoltre, se hai bisogno di osservare diverse proprietà in una struttura di dati nidificata, `watchEffect()` può rivelarsi più efficiente di un watcher avanzato, poiché terrà traccia solo delle proprietà utilizzate nella callback, invece di tracciarle tutte in modo ricorsivo.

:::tip
`watchEffect` monitora le dipendenze solo durante l'esecuzione che avviene in **modo sincrono**. Se lo si utilizza con una callback asincrona, solo le proprietà a cui si accede prima del primo comando `await` vengono considerate come dipendenze.
:::

### `watch` vs. `watchEffect` {#watch-vs-watcheffect}

`watch` e `watchEffect` permettono entrambi di eseguire "side effects" in modo reattivo. La loro differenza principale sta nel modo in cui tracciano le dipendenze reattive:

- `watch` tiene traccia soltanto della fonte specificata che sta osservando. Non tiene conto di ciò che viene utilizzato all'interno della callback. Inoltre, la callback viene innescata solo quando la fonte specifica cambia effettivamente. `watch` separa il monitoraggio delle dipendenze dall'azione che deve essere eseguita, offrendo un controllo più dettagliato sul "quando" la callback deve essere attivata.

- `watchEffect`, invece, unisce il monitoraggio delle dipendenze con l'azione in una sola fase. Esso traccia automaticamente ogni proprietà reattiva che viene utilizzata durante la sua esecuzione sincrona. Questo approccio è più pratico e generalmente porta a un codice più snello, ma rende meno chiare le dipendenze reattive.

</div>

## Side Effect Cleanup {#side-effect-cleanup}

Sometimes we may perform side effects, e.g. asynchronous requests, in a watcher:

<div class="composition-api">

```js
watch(id, (newId) => {
  fetch(`/api/${newId}`).then(() => {
    // callback logic
  })
})
```

</div>
<div class="options-api">

```js
export default {
  watch: {
    id(newId) {
      fetch(`/api/${newId}`).then(() => {
        // callback logic
      })
    }
  }
}
```

</div>

But what if `id` changes before the request completes? When the previous request completes, it will still fire the callback with an ID value that is already stale. Ideally, we want to be able to cancel the stale request when `id` changes to a new value.

We can use the [`onWatcherCleanup()`](/api/reactivity-core#onwatchercleanup) <sup class="vt-badge" data-text="3.5+" /> API to register a cleanup function that will be called when the watcher is invalidated and is about to re-run:

<div class="composition-api">

```js {10-13}
import { watch, onWatcherCleanup } from 'vue'

watch(id, (newId) => {
  const controller = new AbortController()

  fetch(`/api/${newId}`, { signal: controller.signal }).then(() => {
    // callback logic
  })

  onWatcherCleanup(() => {
    // abort stale request
    controller.abort()
  })
})
```

</div>
<div class="options-api">

```js {12-15}
import { onWatcherCleanup } from 'vue'

export default {
  watch: {
    id(newId) {
      const controller = new AbortController()

      fetch(`/api/${newId}`, { signal: controller.signal }).then(() => {
        // callback logic
      })

      onWatcherCleanup(() => {
        // abort stale request
        controller.abort()
      })
    }
  }
}
```

</div>

Note that `onWatcherCleanup` is only supported in Vue 3.5+ and must be called during the synchronous execution of a `watchEffect` effect function or `watch` callback function: you cannot call it after an `await` statement in an async function.

Alternatively, an `onCleanup` function is also passed to watcher callbacks as the 3rd argument<span class="composition-api">, and to the `watchEffect` effect function as the first argument</span>:

<div class="composition-api">

```js
watch(id, (newId, oldId, onCleanup) => {
  // ...
  onCleanup(() => {
    // cleanup logic
  })
})

watchEffect((onCleanup) => {
  // ...
  onCleanup(() => {
    // cleanup logic
  })
})
```

</div>
<div class="options-api">

```js
export default {
  watch: {
    id(newId, oldId, onCleanup) {
      // ...
      onCleanup(() => {
        // cleanup logic
      })
    }
  }
}
```

</div>

`onCleanup` passed via function argument is bound to the watcher instance so it is not subject to the synchronous constraint of `onWatcherCleanup`.

## Tempi di esecuzione della Callback {#callback-flush-timing}

La modifica di uno stato reattivo potrebbe innescare sia gli aggiornamenti del componente Vue sia le callback del watcher da te create.

Similar to component updates, user-created watcher callbacks are batched to avoid duplicate invocations. For example, we probably don't want a watcher to fire a thousand times if we synchronously push a thousand items into an array being watched.

By default, a watcher's callback is called **after** parent component updates (if any), and **before** the owner component's DOM updates. This means if you attempt to access the owner component's own DOM inside a watcher callback, the DOM will be in a pre-update state.

### Post Watchers {#post-watchers}

Se desideri accedere al owner component's DOM in una callback del watcher **dopo** che Vue lo ha aggiornato, devi specificare l'opzione `flush: 'post'`:

<div class="options-api">

```js{6}
export default {
  // ...
  watch: {
    key: {
      handler() {},
      flush: 'post'
    }
  }
}
```

</div>

<div class="composition-api">

```js{2,6}
watch(source, callback, {
  flush: 'post'
})

watchEffect(callback, {
  flush: 'post'
})
```

L'opzione post-flush di `watchEffect()` ha anche un comodo alias, `watchPostEffect()`:

```js
import { watchPostEffect } from 'vue'

watchPostEffect(() => {
  /* eseguito dopo gli aggiornamenti di Vue */
})
```

</div>

### Sync Watchers {#sync-watchers}

It's also possible to create a watcher that fires synchronously, before any Vue-managed updates:

<div class="options-api">

```js{6}
export default {
  // ...
  watch: {
    key: {
      handler() {},
      flush: 'sync'
    }
  }
}
```

</div>

<div class="composition-api">

```js{2,6}
watch(source, callback, {
  flush: 'sync'
})

watchEffect(callback, {
  flush: 'sync'
})
```

Sync `watchEffect()` also has a convenience alias, `watchSyncEffect()`:

```js
import { watchSyncEffect } from 'vue'

watchSyncEffect(() => {
  /* executed synchronously upon reactive data change */
})
```

</div>

:::warning Use with Caution
Sync watchers do not have batching and triggers every time a reactive mutation is detected. It's ok to use them to watch simple boolean values, but avoid using them on data sources that might be synchronously mutated many times, e.g. arrays.
:::

<div class="options-api">

## `this.$watch()` \* {#this-watch}

È anche possibile creare watcher in modo imperativo utilizzando il [metodo di istanza `$watch()`](/api/component-instance#watch):

```js
export default {
  created() {
    this.$watch('question', (newQuestion) => {
      // ...
    })
  }
}
```

Questo è utile quando hai bisogno di configurare un watcher in modo condizionale o quando desideri osservare qualcosa solo in risposta a un'interazione dell'utente. Ti permette anche di fermare il watcher in anticipo.

</div>

## Fermare un Watcher {#stopping-a-watcher}

<div class="options-api">

I watcher dichiarati utilizzando l'opzione `watch` o il metodo di istanza `$watch()` vengono fermati automaticamente quando il componente proprietario passa nello stage `unmounted` (smontato), quindi, nella maggior parte dei casi non devi preoccuparti di fermare tu stesso il watcher.

Nel raro caso in cui hai bisogno di fermare un watcher prima che il componente proprietario venga smontato, l'API `$watch()` restituisce una funzione per farlo:

```js
const unwatch = this.$watch('foo', callback)

// ... quando il watcher non è più necessario:
unwatch()
```

</div>

<div class="composition-api">

I watcher dichiarati in modo sincrono all'interno di `setup()` o `<script setup>` sono legati all'istanza del componente proprietario e verranno fermati automaticamente quando il componente proprietario passa nello stage `unmounted` (smontato). Nella maggior parte dei casi, quindi, non devi preoccuparti di fermare tu stesso il watcher.

Il concetto chiave qui è che il watcher deve essere creato in **modo sincrono**: se il watcher viene creato in una callback asincrona, non sarà legato al componente proprietario e dovrà essere fermato manualmente per evitare perdite di memoria. Ecco un esempio:

```vue
<script setup>
import { watchEffect } from 'vue'

// questo verrà fermato automaticamente
watchEffect(() => {})

// ...questo no!
setTimeout(() => {
  watchEffect(() => {})
}, 100)
</script>
```

Per fermare manualmente un watcher, utilizza la funzione handle. Questo vale sia per `watch` che per `watchEffect`:

```js
const unwatch = watchEffect(() => {})

// ... più tardi, quando non è più necessario
unwatch()
```

Nota che dovrebbero esserci pochissimi casi in cui hai bisogno di creare watcher in modo asincrono, e che la creazione sincrona dovrebbe essere preferita ogni volta che è possibile. Se hai bisogno di attendere alcuni dati asincroni, puoi usare una logica condizionale per il tuo watcher:

```js
// dati da caricare in modo asincrono
const data = ref(null)

watchEffect(() => {
  if (data.value) {
    // fai qualcosa quando i dati sono caricati
  }
})
```

</div>

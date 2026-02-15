---
outline: deep
---

# Usare Vue con TypeScript {#using-vue-with-typescript}

Un sistema di tipi come TypeScript può rilevare molti errori comuni tramite l'analisi in fase di compilazione. Questo riduce le chance di errori in fase di produzione e ci permette di ristrutturare il codice con maggiore sicurezza in grandi applicazioni. TypeScript migliora anche l'ergonomia dello sviluppatore grazie al completamento automatico.

Vue stesso è scritto in TypeScript e offre un supporto TypeScript di prima classe. Tutti i pacchetti ufficiali di Vue includono dichiarazioni di tipi integrate che dovrebbero funzionare senza problemi.

## Creare un applicazione {#project-setup}

[`create-vue`](https://github.com/vuejs/create-vue), il tool ufficiale, offre la possibilità di creare un progetto Vue [Vite](https://vite.dev/)-powered, TypeScript-ready.

### Overview {#overview}

Con una configurazione basata su Vite, il server di sviluppo e il bundler eseguono solo la transpilazione e non effettuano alcun controllo dei tipi. Questo assicura che il server di sviluppo di Vite rimanga incredibilmente veloce anche quando si utilizza TypeScript.

- Durante lo sviluppo, raccomandiamo di affidarsi a una buona [configurazione dell'IDE](#ide-support) per eventuali errori istantanei.

- Se si utilizzano gli SFC, è possibile utilizzare [`vue-tsc`](https://github.com/vuejs/language-tools/tree/master/packages/tsc) per il controllo dei tipi da linea di comando e per la generazione delle dichiarazioni di tipo. `vue-tsc` è un wrapper attorno a `tsc`, l'interfaccia della riga di comando di TypeScript. Funziona in gran parte allo stesso modo di `tsc` ma supporta anche i file SFC di Vue oltre ai file TypeScript. È possibile eseguire `vue-tsc` in modalità di watch in parallelo con il server di sviluppo Vite, oppure utilizzare un suo plugin come [vite-plugin-checker](https://vite-plugin-checker.netlify.app/) che esegue i controlli in un thread separato.

- Vue CLI offre anche il supporto per TypeScript, ma è stata deprecata. Vedere le [note di seguito](#note-on-vue-cli-and-ts-loader).

### Supporto degli IDE {#ide-support}

- [Visual Studio Code](https://code.visualstudio.com/) (VS Code) è fortemente raccomandato per il suo eccellente supporto "out-of-the-box" per TypeScript.

  - [Vue - Official](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (precedentemente Volar) è l'estensione ufficiale di VS Code che fornisce il supporto TypeScript all'interno dei file SFC di Vue, insieme a molte altre fantastiche funzionalità.

    :::tip
    Vue - Official extension sostituisce [Vetur](https://marketplace.visualstudio.com/items?itemName=octref.vetur), la nostra precedente estensione ufficiale di VS Code per Vue 2. Se hai attualmente Vetur installato, assicurati di disabilitarlo nei progetti di Vue 3.
    :::

- [WebStorm](https://www.jetbrains.com/webstorm/) fornisce anche un supporto nativo sia per TypeScript che per Vue. Altri IDE JetBrains li supportano anche, sia nativamente che tramite [un plugin gratuito](https://plugins.jetbrains.com/plugin/9442-vue-js). As of version 2023.2, WebStorm and the Vue Plugin come with built-in support for the Vue Language Server. You can set the Vue service to use Volar integration on all TypeScript versions, under Settings > Languages & Frameworks > TypeScript > Vue. By default, Volar will be used for TypeScript versions 5.0 and higher.

### Configurare `tsconfig.json` {#configuring-tsconfig-json}

I progetti creati tramite `create-vue` includono un file `tsconfig.json`. La configurazione di base è astratta nel pacchetto [`@vue/tsconfig`](https://github.com/vuejs/tsconfig). All'interno del progetto, utilizziamo i [riferimenti di progetto](https://www.typescriptlang.org/docs/handbook/project-references.html) per garantire tipi corretti per il codice che viene eseguito in diversi ambienti (ad esempio, il codice dell'applicazione e il codice di test dovrebbero avere variabili globali diverse).

Quando si configura manualmente il file `tsconfig.json`, alcune opzioni importanti includono:

- [`compilerOptions.isolatedModules`](https://www.typescriptlang.org/tsconfig#isolatedModules) è impostata su `true` perché Vite usa [esbuild](https://esbuild.github.io/) per la transpilazione di TypeScript ed è soggetto a limitazioni di transpilazione per file singoli. [`compilerOptions.verbatimModuleSyntax`](https://www.typescriptlang.org/tsconfig#verbatimModuleSyntax) is [a superset of `isolatedModules`](https://github.com/microsoft/TypeScript/issues/53601) and is a good choice, too - it's what [`@vue/tsconfig`](https://github.com/vuejs/tsconfig) uses.

- Se stai utilizzando l'Options API, è necessario impostare  [`compilerOptions.strict`](https://www.typescriptlang.org/tsconfig#strict) su `true` (o almeno abilitare [`compilerOptions.noImplicitThis`](https://www.typescriptlang.org/tsconfig#noImplicitThis), che fa parte del flag `strict`) per sfruttare il controllo dei tipi di `this` nelle opzioni del componente. In caso contrario, `this` verrà considerato come `any`.

- Se hai configurato alias di risoluzione nel tuo strumento di compilazione, ad esempio l'alias  `@/*` configurato per impostazione predefinita in un progetto `create-vue`, devi configurarlo anche per TypeScript tramite [`compilerOptions.paths`](https://www.typescriptlang.org/tsconfig#paths).

- If you intend to use TSX with Vue, set [`compilerOptions.jsx`](https://www.typescriptlang.org/tsconfig#jsx) to `"preserve"`, and set [`compilerOptions.jsxImportSource`](https://www.typescriptlang.org/tsconfig#jsxImportSource) to `"vue"`.

Controlla anche:

- [Opzioni del compilatore ufficiale TypeScript](https://www.typescriptlang.org/docs/handbook/compiler-options.html)
- [Caveats sulla compilazione di TypeScript con esbuild](https://esbuild.github.io/content-types/#typescript-caveats)

### Note sulla Vue CLI e `ts-loader` {#note-on-vue-cli-and-ts-loader}

Nei setup basati su Webpack come Vue CLI, è comune eseguire il controllo dei tipi come parte della pipeline di trasformazione dei moduli, ad esempio con `ts-loader`. Tuttavia, questa non è una soluzione pulita perché il sistema di tipi ha bisogno di conoscere l'intero grafo dei moduli per eseguire i controlli dei tipi. La fase di trasformazione di singoli moduli semplicemente non è il posto giusto per questa attività e ciò porta ai seguenti problemi:

- `ts-loader` può effettuare il controllo dei tipi solo dopo la trasformazione del codice. Questo non è in linea con gli errori che vediamo negli IDE o da `vue-tsc`, che vengono mappati direttamente al codice sorgente.

- Il controllo dei tipi può essere lento. Quando viene eseguito nello stesso thread/processo delle trasformazioni del codice, influisce significativamente sulla velocità di compilazione dell'intera applicazione.

- Già abbiamo il controllo dei tipi in esecuzione direttamente nel nostro IDE in un processo separato, quindi il costo di rallentamento dell'esperienza di sviluppo semplicemente non è un buon compromesso.

Se stai attualmente utilizzando Vue 3 + TypeScript tramite Vue CLI, ti consigliamo vivamente di migrare a Vite. Stiamo inoltre lavorando su opzioni CLI per abilitare il supporto TS solo per la compilazione, in modo che tu possa passare a `vue-tsc` per il controllo dei tipi.

## Note generali sull'utilizzo {#general-usage-notes}

### `defineComponent()` {#definecomponent}

Per consentire a TypeScript di inferire correttamente i tipi all'interno delle opzioni del componente, è necessario definire i componenti utilizzando [`defineComponent()`](/api/general#definecomponent):

```ts
import { defineComponent } from 'vue'

export default defineComponent({
  // type inference enabled
  props: {
    name: String,
    msg: { type: String, required: true }
  },
  data() {
    return {
      count: 1
    }
  },
  mounted() {
    this.name // tipo: string | undefined
    this.msg // tipo: string
    this.count // tipo: number
  }
})
```

`defineComponent()` può anche ottenere le props passate a `setup()` quando si utilizza la Composition API senza `<script setup>`:

```ts
import { defineComponent } from 'vue'

export default defineComponent({
  // type inference enabled
  props: {
    message: String
  },
  setup(props) {
    props.message // tipo: string | undefined
  }
})
```

Guarda anche:

- [Note sul webpack Treeshaking](/api/general#note-on-webpack-treeshaking)
- [Test dei tipi per `defineComponent`](https://github.com/vuejs/core/blob/main/packages-private/dts-test/defineComponent.test-d.tsx)

:::tip
`defineComponent()` abilita anche l'inferring dei tipi per i componenti definiti in JavaScript puro.
:::

### Utilizzo nei Single-File Components {#usage-in-single-file-components}

Per utilizzare TypeScript nei SFC (Single-File Components), aggiungi l'attributo `lang="ts"` ai tag `<script>`. Quando `lang="ts"` è presente, tutte le espressioni del template vengono sottoposte a un controllo di tipo più rigoroso.

```vue
<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  data() {
    return {
      count: 1
    }
  }
})
</script>

<template>
  <!-- type checking e auto-completion abilitati -->
  {{ count.toFixed(2) }}
</template>
```

`lang="ts"` può anche essere utilizzato con `<script setup>`:

```vue
<script setup lang="ts">
// TypeScript abilitato
import { ref } from 'vue'

const count = ref(1)
</script>

<template>
  <!-- type checking e auto-completion abilitati -->
  {{ count.toFixed(2) }}
</template>
```

### TypeScript nei Templates {#typescript-in-templates}

Il tag `<template>` supporta anche TypeScript nelle espressioni di binding quando viene utilizzato `<script lang="ts">` o `<script setup lang="ts">`. Questo è utile nei casi in cui è necessario effettuare il type casting nelle espressioni del template.

Ecco un esempio:

```vue
<script setup lang="ts">
let x: string | number = 1
</script>

<template>
  <!-- errore perché x potrebbe essere una stringa -->
  {{ x.toFixed(2) }}
</template>
```

Questo può essere risolto con un type cast inline:

```vue{6}
<script setup lang="ts">
let x: string | number = 1
</script>

<template>
  {{ (x as number).toFixed(2) }}
</template>
```

:::tip
Se si utilizza Vue CLI o una configurazione basata su webpack, TypeScript nelle espressioni dei template richiede `vue-loader@^16.8.0`.
:::

### Utilizzo con TSX {#usage-with-tsx}

Vue supporta anche la creazione di componenti con JSX / TSX. I dettagli sono trattati nella guida [Render Function & JSX](/guide/extras/render-function.html#jsx-tsx).

## Componenti generici {#generic-components}

I componenti generici sono supportati in due casi:

- Nei SFC: [`<script setup>` con l'attributo `generic`](/api/sfc-script-setup.html#generics)
- Render function / JSX components: [`defineComponent()`](/api/general.html#function-signature)

## API-Specific {#api-specific-recipes}

- [TS con Composition API](./composition-api)
- [TS con Options API](./options-api)

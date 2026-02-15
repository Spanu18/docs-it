---
sidebar: false
ads: false
editLink: false
sponsors: false
---

<script setup>
import SponsorsGroup from '@theme/components/SponsorsGroup.vue'
import { load, data } from '@theme/components/sponsors'
import { onMounted } from 'vue'

onMounted(load)
</script>

# Diventa uno sponsor di Vue.js {#become-a-vue-js-sponsor}

Vue.js è un progetto open source con licenza MIT ed il suo uso è completamente gratuito.
Il notevole sforzo per mantenere un ecosistema così grande e sviluppare nuove funzionalità per il progetto è reso possibile solo grazie al generoso sostegno finanziario dei nostri sponsor.

## Come diventare Sponsor {#how-to-sponsor}

Le sponsorizzazioni possono essere effettuate tramite [GitHub Sponsors](https://github.com/sponsors/yyx990803) o [OpenCollective](https://opencollective.com/vuejs). Le fatture possono essere ottenute tramite il sistema di pagamento di GitHub. Sono accettate sia le donazioni ricorrenti mensili che le donazioni una tantum. Le sponsorizzazioni ricorrenti danno diritto all'inserimento del proprio logo come specificato in [Livelli di Sponsorizzazione](#tier-benefits).

Se hai domande riguardo ai livelli, alla logistica dei pagamenti o alle informazioni sulla visibilità della sponsorizzazione, non esitare a contattarci a [sponsor@vuejs.org](mailto:sponsor@vuejs.org?subject=Vue.js%20sponsorship%20inquiry).

## Sponsorizzare Vue come azienda {#sponsoring-vue-as-a-business}

Sponsorizzare Vue ti offre una grande visibilità presso oltre **2 milioni** di sviluppatori Vue in tutto il mondo attraverso il nostro sito web e i README del nostro progetto su GitHub. Questo non solo genera lead direttamente, ma migliora anche la riconoscibilità del tuo brand come azienda attenta all'Open Source. Si tratta di una risorsa intangibile ma estremamente importante per le aziende che realizzano prodotti per sviluppatori, poiché migliora il tasso di conversione.

Se stai utilizzando Vue per creare un prodotto che genera entrate, ha senso dal punto di vista commerciale sponsorizzare lo sviluppo di Vue: **assicura che il progetto su cui il tuo prodotto si basa rimanga solido e attivamente mantenuto.** L'esposizione e l'immagine positiva del tuo brand nella comunità Vue facilitano anche l'attrazione e il reclutamento di sviluppatori Vue.

Se stai realizzando un prodotto il cui target di clienti sono gli sviluppatori, otterrai traffico di alta qualità grazie alla sponsorizzazione, dato che tutti i nostri visitatori sono sviluppatori. La sponsorizzazione contribuisce anche a costruire il riconoscimento del brand e a migliorare la conversione.

## Sponsorizzare Vue come privato {#sponsoring-vue-as-an-individual}

Se sei un privato e ti sei trovato bene usando Vue, considera di fare una donazione come segno di apprezzamento - come offrirci un caffè di tanto in tanto. Molti dei nostri membri del team accettano sponsorizzazioni e donazioni tramite GitHub Sponsors. Cerca il pulsante "Sponsor" sul profilo di ciascun membro del team sulla nostra [pagina del team](/about/team).

Puoi anche cercare di convincere il tuo datore di lavoro a sponsorizzare Vue come azienda. Questo potrebbe non essere facile, ma le sponsorizzazioni aziendali hanno generalmente un impatto molto più grande sulla sostenibilità dei progetti OSS rispetto alle donazioni individuali, quindi ci aiuterai molto di più se avrai successo.

## Benefici dei livelli {#tier-benefits}

- **Sponsor speciale globale**:
  - Limitato a **uno** sponsor a livello globale. <span v-if="!data?.special">Attualmente vacante. [Mettiti in contatto](mailto:sponsor@vuejs.org?subject=Vue.js%20special%20sponsor%20inquiry)!</span><span v-else>(Attualmente occupato)</span>
  - (Esclusivo) Posizionamento del logo **above the fold** sulla prima pagina di [vuejs.org](/).
  - (Esclusivo) Ringraziamenti speciali e retweet regolari dei principali lanci di prodotti tramite l' [l'account ufficiale di Vue su X](https://x.com/vuejs) (320k followers).
  - Posizionamento del logo più visibile in tutte le posizioni dei livelli inferiori.
- **Platino (USD$2,000/mo)**:
  - Inserimento di un logo in risalto sulla pagina principale di [vuejs.org](/).
  - Inserimento di un logo in risalto nella sidebar di tutte le pagine di contenuto.
  - Inserimento di un logo in risalto nel README di [`vuejs/core`](https://github.com/vuejs/core) e [`vuejs/vue`](https://github.com/vuejs/core).
- **Oro (USD$500/mo)**:
  - Inserimento di un logo Grande sulla pagina principale di [vuejs.org](/).
  - Inserimento di un logo Grande nel README di `vuejs/core` e `vuejs/vue`.
- **Argento (USD$250/mo)**:
  - Inserimento di un logo medio nel file `BACKERS.md` di `vuejs/core` e `vuejs/vue`.
- **Bronzo (USD$100/mo)**:
  - Inserimento di un logo piccolo nel file `BACKERS.md` di `vuejs/core` e `vuejs/vue`.
- **Sostenitore generoso (USD$50/mo)**:
  - Nome elencato nel file `BACKERS.md` di `vuejs/core` e `vuejs/vue`, sopra gli altri sostenitori individuali.
- **Sostenitore individuale (USD$5/mo)**:
  - Nome elencato nel file `BACKERS.md` di `vuejs/core` e `vuejs/vue`.

## Sponsor attuali {#current-sponsors}

### Sponsor Global Special {#special-global-sponsor}

<SponsorsGroup tier="special" placement="page" />

### Platinum {#platinum}

<SponsorsGroup tier="platinum" placement="page" />

### Platinum (China) {#platinum-china}

<SponsorsGroup tier="platinum_china" placement="page" />

### Gold {#gold}

<SponsorsGroup tier="gold" placement="page" />

### Silver {#silver}

<SponsorsGroup tier="silver" placement="page" />

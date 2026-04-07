# Prefazione

## Chi scrive

Mi chiamo Alessandro Osti, faccio il programmatore da 25 anni. Non il tipo da palcoscenico alle conferenze tech — il tipo che scrive codice, lo mette in produzione, e poi ci convive nel tempo. Lavoro come consulente backend in ambito bancario e assicurativo, principalmente su stack Java/Oracle, e negli ultimi anni sono migrato su Python per i miei progetti personali.

Il progetto di cui parlo in questo libro si chiama ARIA. È un sistema di analisi finanziaria — un chatbot che fa analisi ciclica, calcola canali, interroga modelli LLM per generare risposte, e serve il tutto attraverso un'interfaccia web. Il backend è in Python con FastAPI, il frontend in Vue 3 con Tailwind. Non è un esercizio accademico — è in produzione su maotrade.it, la mia piattaforma personale di trading che uso ogni giorno.

MAOTrade esiste dal 2012. L'ho costruita insieme a mio padre, che era un direttore di banca con la passione per l'analisi ciclica. ARIA è nata da quella base.

ARIA è stata sviluppata in gran parte con Claude Code. Questo libro racconta come.

Non sono un evangelista dell'intelligenza artificiale, non vendo corsi, non ho un canale YouTube. Sono uno sviluppatore che studia l'AI come strumento — prima attraverso un percorso strutturato sui fondamentali degli LLM, poi costruendo ARIA come caso reale. Ho trovato un modo di lavorare che funziona e ho deciso di raccontarlo partendo dai dati — le sessioni reali, i prompt reali, i costi reali, gli errori reali. Comprese le volte in cui le cose sono andate male.

## Cosa troverai in questo libro

**Capitolo 1 — Da Claude.ai a Claude Code.** Come lavoravo prima di Claude Code, incollando codice avanti e indietro tra il browser e l'editor. Cosa è cambiato con l'arrivo dello strumento, e cosa sorprendentemente non è cambiato. Perché il mio ruolo si è trasformato da sviluppatore a team leader.

**Capitolo 2 — I fondamentali.** I mattoni di base imparati dal corso di Anthropic su DeepLearning.ai: la `@` per dare contesto, "Think Hard" per forzare il ragionamento, il CLAUDE.md per dare memoria al progetto. Cosa sono, come funzionano, e perché da soli non bastano.

**Capitolo 3 — Il metodo.** Le cinque fasi emerse dal lavoro quotidiano — contesto, vincolo, diagnosi, implementazione, verifica — raccontate con esempi reali dalle sessioni. Come si scrive un piano di implementazione che funziona come prompt. Come si sfidano le risposte sbagliate. Come si usa la documentazione per far comunicare due progetti che vivono in repository separati.

**Capitolo 4 — Backend vs Frontend: stesso metodo, leve diverse.** Come cambia il modo di usare Claude Code quando la competenza tecnica è diversa. Sul backend faccio l'architetto, sul frontend faccio il committente. I numeri delle sessioni confermano la differenza — e mostrano cosa costa in più delegare.

**Capitolo 5 — Quando il metodo salta.** Le sessioni in cui le cose sono andate storte. Un deployment che non partiva, una documentazione che non corrispondeva al codice. Cosa succede quando abbassi la guardia, e perché il metodo serve di più proprio quando ti senti meno sicuro.

**Capitolo 6 — I numeri e le conclusioni.** Quaranta sessioni, centotrenta dollari, un sistema in produzione. Cosa dicono i dati sulla realtà del vibe coding, cosa non dicono, e perché la competenza resta l'ingrediente che fa la differenza tra un prodotto con una direzione e un prodotto vuoto.

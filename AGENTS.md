# AGENTS.md — Playbook (istruzioni di lavoro)

> Questo file è la **memoria del progetto**. Aprilo all'inizio di ogni sessione.
> Scopo: scrivere una pagina (o poche) per volta, con qualità costante, senza ripartire da zero.

> **Due playbook, stesse regole.** Le istruzioni qui valgono identiche per entrambi i percorsi:
> - **AI Playbook** — contenuto in `ai/`, syllabus in [`ai/SYLLABUS.md`](ai/SYLLABUS.md), stato pagine in [`ai/README.md`](ai/README.md).
> - **Cloud Playbook** — contenuto in `cloud/`, syllabus in [`cloud/SYLLABUS.md`](cloud/SYLLABUS.md), stato pagine in [`cloud/README.md`](cloud/README.md).
>
> Stessa voce, stesso template di lezione, stessi badge di stato. Studio in alternanza a blocchi (mai due fronti aperti lo stesso giorno). Il RADAR è unico in [`ai/RADAR.md`](ai/RADAR.md).

---

## 1. Cos'è questo progetto

Una guida di studio sull'AI applicativa moderna, in Markdown, per **diventare competitivi nel campo senza laurea** — capire di cosa parlano tutti, saper decidere, saper costruire l'essenziale.

- **Pubblico**: io. Background: so programmare bene (Python/SW eng), niente ML formale.
- **Obiettivo a tendere**: profilo da AI Engineer applicativo → AI Solution Architect.
- **Non interessa**: addestramento di modelli da zero, ricerca, matematica pesante.
- **Interessa**: progettare, valutare, mettere in sicurezza, deployare e governare sistemi basati su LLM.
- **Motore di pubblicazione**: Markdown puro. Studiabile in Obsidian, pubblicabile con MkDocs Material o Docusaurus (decisione reversibile, non bloccante).

## 2. Principio guida (non violarlo)

**Organizzazione per PROBLEMA, non per tecnologia.** I tool muoiono in fretta; i problemi restano. Ogni pagina risponde a "quale decisione/bisogno reale copro", non "elenco di feature di un tool".

**Profondità selettiva.** Meglio padroneggiare il 30% giusto che spalmarsi sul 100%. Il taglio è "deciso sul design (da architetto), pratico sull'essenziale (codice solo dove va toccato con mano)".

**Il contenuto vince sul contenitore.** Non perdere tempo su tooling/estetica: il valore è nelle pagine.

## 3. Template di ogni pagina (LEZIONE) — versione definitiva

> Riferimenti di qualità: `ai/foundations/come-funziona-un-llm.md` (0.1) e
> `ai/foundations/embedding.md` (0.2). La 0.1 è il **gold standard di densità**
> (tabella anti-pattern inclusa); la 0.2 è il pavimento accettabile.
> Il template è una CASSETTA DEGLI ATTREZZI, non una checklist da riempire sempre tutta.
> Usa giudizio: su una lezione semplice alcuni blocchi si accorpano o si saltano. Riempire
> tutti i blocchi a forza = rumore. Il valore è usarne pochi al momento giusto.

Ordine dei blocchi (NESSUN emoji nei titoli di sezione — prosa pulita):

1. **Intestazione "contratto"** — frontmatter (`title`, `sidebar_position`) + H1 + `<div class="lesson-meta">` con badge di stato, numero lezione (es. "Lezione 0.2"), durata stimata + `<p class="lesson-lead">` con la frase chiave in linguaggio COMPRENSIBILE PRIMA di leggere (mai termini ancora da definire: NON "saprai regolare la temperature" prima di averla spiegata).
2. **Apertura** — attacco diretto: la domanda lasciata in sospeso dalla lezione precedente, o un'osservazione concreta. Niente sezione "Prima di leggere" formale: l'aggancio sta nella prosa.
3. **Idea in una frase + ancora mentale** — il cuore del concetto in una riga + analogia, integrata nel discorso (non in un box separato).
4. **Corpo — i mattoni** — i concetti sviluppati in prosa narrativa profonda. Una sotto-sezione (H2) per ogni mattone. Dettagli su voce e profondità in §5.
5. **Approfondimento a click** — per i termini che presuppongono prerequisiti: il termine corretto + una riga minima autosufficiente nel flusso, poi un blocco `<details>` espandibile per chi vuole scendere. Chi sa salta, chi non sa apre. NON introdurre nel blocco altri termini che richiederebbero a loro volta una spiegazione: rimanda quelli alla loro lezione.
6. **Mermaid** — SOLO se la struttura è visiva. Saltabile su lezioni che non lo richiedono.
7. **Sotto il cofano** — NON opzionale quando c'è una formula o un algoritmo dietro. Prima la matematica/algoritmo VERO (formula corretta, termini esatti), POI la traduzione intuitiva (cosa SIGNIFICA, non "salta la matematica"). Vedi §5.
8. **Cosa NON è** — tabella "pensiero sbagliato → come stanno le cose". Vedi 0.1 e 0.2 come riferimento.
9. **Cosa dura / cosa evitare** — dove serve, con i badge di stato (§4). Isola il deperibile.
10. **Verifica di comprensione** — domande di recupero a memoria + "rivedi DOMANI ciò che hai sbagliato" (spacing). Inquadrare lo sforzo come positivo, non come fallimento. Le ultime 1-2 domande possono anticipare lezioni future.
11. **Glossario della pagina** — i termini di QUESTA pagina, in fondo.
12. **Per approfondire** — cosa cercare e dove. MAI link/titoli/numeri di pagina inventati. URL precisi solo se verificati via web search.
13. **Prossima lezione** — gancio in prosa breve che mostra come il concetto apre la successiva.

## 4. Badge di stato (meccanismo anti-fuffa)

- `<span class="badge-stato stabile">Stabile</span>` — regge per anni
- `<span class="badge-stato evoluzione">In evoluzione</span>` — cambierà
- `<span class="badge-stato rischio">A rischio</span>` — potrebbe morire
- `<span class="badge-stato legacy">Legacy</span>` — citato solo per contesto

## 5. Tono e regole di scrittura (DISTILLATE DAL FEEDBACK — vincolanti)

**Lingua:** italiano.

**VOCE DEFINITIVA (calibrata con feedback reale — questo è IL registro):**
Scrivi come un collega giovane e sveglio che spiega bene a voce — la stessa voce dei messaggi
di chat ben scritti, NON un manuale in posa. Caratteristiche precise:
- **Diretto, va al sodo.** Niente preamboli, niente costruzioni che ritardano il senso per fare
  effetto (es. evita "le regole — tantissime"). La frase consegna il contenuto subito.
- **Frasi brevi e dirette, con stop di ritmo** — ma OGNI frase corta deve guadagnarsi il punto:
  o apre una tensione o chiude un colpo. Se ripete soltanto, è grasso travestito da ritmo: toglila.
- **Scorrevole ≠ impastato.** "Scorrevole" significa togliere gli inciampi, NON legare tutto in
  periodi lunghi. Mantieni gli stop voluti (es. "La differenza è grossa.").
- **Grassetti funzionali** come appigli per l'occhio: sul perno del concetto in quella frase, così
  chi legge in fretta o si distrae trova il filo. Dosati: se è tutto grassetto, niente lo è.
- **Concreto, sempre.** Termine astratto ("regole") → esempio vivo la prima volta (lo spam scritto
  a mano vs gli esempi etichettati). Mai astrazione nuda.
- **Battute dosate.** Un guizzo ogni tanto, SOLO dove il concetto lo regge (es. "il poeta dopo il
  secondo bicchiere", "statistica che si è messa il vestito buono"). MAI a ogni paragrafo: stanca
  e diventa l'opposto. Se un paragrafo non la regge, niente battuta.
- **Per tutti.** Un ventenne e un sessantenne lo capiscono uguale. Niente gergo accademico
  ("inferisce" → "ricava/tira fuori"), niente infantilizzazione.
- **Si fida del lettore.** Niente meta-istruzioni nel corpo ("dillo a voce", "questo fissa il
  concetto"): quelle vivono nei BOX (curiosità/nota/approfondisci) e nella verifica finale.

**Riferimento vivo:** `foundations/come-funziona-un-llm.md` (1.1) è il gold standard di questa voce.

**Voce:** via di mezzo tra "collega senior che ti spiega davanti a un caffè" e "docente
brillante": tecnica e precisa, scorrevole, narrativa, con qualche guizzo di personalità
(es. analogie "notaio/poeta") MA senza fare il buffone. Parla direttamente al lettore-sviluppatore.

**Profondità:** ALTA. Spiega davvero, sviluppa il concetto, rendi sempre esplicito il PERCHÉ.
Non accontentarti di definizioni atomizzate: il lettore vuole CAPIRE, non spuntare.

**Lunghezza:** NESSUN limite massimo. Regola d'oro: *chiarezza > lunghezza > brevità*. Espandi
finché il concetto non è limpido, poi FERMATI. Mai allungare per allungare: scrivere lungo per
dire le stesse cose è vietato. Ogni paragrafo deve guadagnarsi il posto aggiungendo comprensione.

**Analogie:** integrate NEL discorso (prosa), non in box separati. Devono affiancare la
precisione, non sostituirla (un'analogia semplifica e a volte "mente" un po': non farla passare
per la verità tecnica completa).

**Linguaggio tecnico:** usa SEMPRE il termine tecnico vero e completo (es. "chunk", non "pezzo";
"context window"). Il lettore mastica bene il tecnico: non annacquare.

**REGOLA FERREA — acronimi e termini:**
- Sciogli OGNI acronimo/sigla alla PRIMA occorrenza, sempre (es. "LLM — Large Language Model").
  Nessuna eccezione, nemmeno per termini del titolo.
- Spiega ogni termine tecnico alla prima occorrenza, inline, nel punto in cui appare
  (NON in un glossario a inizio pagina: le definizioni fuori contesto non si fissano).
- Il glossario va in FONDO, come riferimento di ripasso.

**REGOLA DI COPERTURA (syllabus):** ogni termine tecnico deve essere spiegato DA QUALCHE PARTE
nel percorso, al momento giusto secondo le dipendenze. Non serve spiegarlo in ogni pagina dove
appare, ma deve essere coperto PRIMA che diventi indispensabile. Non introdurre in una pagina
concetti che appartengono a una lezione successiva (es. niente "chunk" nella lezione sul
funzionamento dell'LLM: i chunk sono roba di RAG, lezione 2.1). Token ≠ chunk: non confonderli.

**Termini con prerequisiti:** quando il termine corretto presuppone altri concetti (es.
"modello" presuppone parametri/addestramento/rete neurale), NON scappare verso una parola
semplice ma imprecisa (es. "macchina"). Usa il termine corretto + spiegazione minima
autosufficiente inline, e metti l'approfondimento in un blocco a click (vedi template 4b).
Dosare la profondità: di' quel tanto che serve ORA, rimanda il resto alla lezione che lo copre
(es. "rete neurale" e "transformer" non si spiegano nella lezione 1.1, si nominano e si rimandano).
Evita metafore imprecise quando esiste il termine tecnico: usa "modello" per l'LLM, non "macchina".

**"Sotto il cofano":** tecnico vero PRIMA (formula reale con termini corretti), traduzione
intuitiva DOPO (cosa significa quella matematica senza doverla calcolare). Obiettivo: il lettore
CAPISCE la formula, non la evita.

**Fonti:** mai inventare link, titoli di video, o numeri di pagina. Indicare "cosa cercare e
dove". URL esatti solo se verificati con ricerca web.

**Codice (per le pagine che lo richiedono):** snippet brevi e illustrativi, solo dove serve
averlo toccato con mano. Mai progetti completi.

## 6. Struttura e stato

La struttura completa e l'ordine di studio con i prerequisiti vivono in **`ai/SYLLABUS.md`**
(8 parti, 0-7, più capstone). Lo stato di ogni pagina (scritta / da riscrivere / bozza) vive in
**`ai/README.md`** ed è il punto di verità — da aggiornare quando una bozza diventa scritta.

AGENTS.md NON duplica la lista delle pagine: due fonti uguali divergono presto. Qui restano
solo le regole di scrittura; cosa scrivere si legge dal SYLLABUS, in che stato è si legge
dal README.

Reference di qualità (gold standard della voce, da rileggere prima di scrivere una lezione nuova):
- `ai/foundations/come-funziona-un-llm.md` (0.1) — densità piena, tabella anti-pattern.
- `ai/foundations/embedding.md` (0.2) — pavimento accettabile.

## 7. Decision drill (formato)

Una pagina per macroarea. Struttura: **Scenario** realistico (con vincoli: budget, latenza, dati) → "decidi e giustifica" → **griglia di valutazione** (cosa nominerebbe un senior, trade-off attesi, trappole). Lo scopo è allenare il giudizio, non la memoria.

## 8. Il Radar (meccanismo di aggiornamento)

NON un news feed. Traccia solo i **cambiamenti strutturali**: quando qualcosa passa da curiosità a standard de facto e merita una pagina. Stati: `EMERGENTE` → `IN ADOZIONE` → `STANDARD` (esce dal radar, diventa pagina) → `LEGACY`. Esempio già applicato: MCP, promosso a STANDARD → pagina creata. Il giudizio è UMANO, niente automazione cieca.

## 9. Come lavorare a pezzi (workflow per sessione)

1. Apri questo AGENTS.md.
2. Apri `ai/README.md` per vedere lo stato; scegli **1-3 pagine** in bozza (non di più: la qualità cala).
3. Per ognuna: segui il template (§3), il tono (§5), i badge (§4). Usa `foundations/come-funziona-un-llm.md` (0.1) e `foundations/embedding.md` (0.2) come metro di qualità.
4. Aggiorna lo stato in `ai/README.md` ("bozza" → "scritta").
5. Se qualcosa è "diventato standard", aggiorna il Radar e il Changelog (quando esisteranno).
6. Fine sessione: salva. La prossima riprende da qui.

### Priorità suggerita (per imparare nell'ordine giusto)
1. Completa la Parte 0 (Fondamenta) — sono i prerequisiti di tutto.
2. Coppie chiuse Parte 1 ⇄ Parte 3 (vedi SYLLABUS): costruisci un pezzo, valutalo, poi avanti.
3. Parte 2 (Multimodale) quando vuoi il quadro completo di cosa sa fare l'AI.
4. Parti 4, 5, 6, 7 quando un progetto reale te le chiede.

## 10. Anti-pattern di questo progetto (imparati sul campo)

Da evitare:
- Discutere all'infinito tooling/struttura invece di scrivere contenuti.
- Voler fare tutte le pagine in una volta (escono scadenti).
- Aggiungere automazioni "magiche" (cron che depreca da solo) — l'AI inventa, serve l'umano nel loop.
- Linkare immagini esterne, organizzare per tool invece che per problema.

Da fare:
- Una pagina alla volta, fatta bene, con active recall. Poi un progetto pratico.

# Prompt per scrivere una lezione della AI Playbook

> **Come si usa**
>
> 1. Apri il SYLLABUS e individua la lezione da scrivere (numero, titolo, scope, dipendenze, posizione).
> 2. Compila i campi del **blocco VARIABILE** qui sotto (i `{{PLACEHOLDER}}`). Non toccare il blocco FISSO.
> 3. Copia l'intero prompt (variabile + fisso) e incollalo a Sonnet (o altro modello strong) in una conversazione che abbia accesso al filesystem del repo.
> 4. Quando consegna, valutala con la stessa rubric usata per 0.1/0.2 (densità, voce, tabella anti-pattern, sotto-il-cofano, bridge).
>
> **Quando NON usare questo prompt:** per lezioni-pratiche di codice (es. "Costruisci un RAG minimale") o per le **decision drill** (formato diverso, vedi AGENTS.md §7). Per quelle scrivi un prompt dedicato.

---

## BLOCCO VARIABILE — da compilare per ogni lezione

```
LEZIONE: {{numero}} — {{titolo}}
FILE: ai/{{cartella}}/{{slug}}.md

SCOPE (incollato letterale dal SYLLABUS, riga ~{{N}}):
{{blocco SYLLABUS verbatim}}

CONFINE NETTO (cosa NON entra in questa lezione, e dove vive invece):
- {{cosa1}} — vive in {{lezione X}}
- {{cosa2}} — vive in {{lezione Y}}

AGGANCIO IN APERTURA (filo lasciato dalla lezione {{precedente}}):
{{una-due frasi: la domanda/osservazione in sospeso da cui partire}}

BRIDGE IN CHIUSURA (verso la lezione {{successiva}}):
{{una-due frasi: come questa lezione apre la prossima}}

CROSS-REFERENCE VERIFICATI (solo lezioni che esistono nel SYLLABUS):
- {{lezione A}} — quando parli di {{argomento}}
- {{lezione B}} — quando parli di {{argomento}}
- {{...}}

CANDIDATI PER I MATTONI H2 (l'agente è libero di adattarli, ma è il punto di partenza):
1. {{mattone 1}}
2. {{mattone 2}}
3. {{mattone 3}}
4. {{mattone 4}}

SOTTO IL COFANO (se c'è una formula/algoritmo, qui va indicato quale):
- {{formula o algoritmo, es. "discesa del gradiente: θ ← θ − η ∇L(θ)"}}
- Posizionamento suggerito: {{H3 in linea (stile 0.1) / <details> collassato (stile 0.2)}}
  Linea guida: H3 in linea se è centrale alla spiegazione; <details> se è un approfondimento opzionale.

TABELLA "COSA NON È" — 4 misconception candidate (l'agente può sostituire se ne trova di migliori):
| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "{{misc1}}" | {{realtà1}} |
| "{{misc2}}" | {{realtà2}} |
| "{{misc3}}" | {{realtà3}} |
| "{{misc4}}" | {{realtà4}} |

BADGE DI STATO (uno tra: stabile / evoluzione / rischio / legacy):
{{stato}}

DURATA STIMATA: ~{{N}} min di lettura
```

---

## BLOCCO FISSO — vale per ogni lezione, non modificare

Sei un agente che scrive una lezione della "AI Playbook", in italiano. Il repo è una guida di studio sull'AI applicativa basata su Docusaurus. Il file da scrivere è quello indicato in `FILE` sopra: lo **scaffold esiste già** (frontmatter, `<div class="lesson-meta">`, `<p class="lesson-lead">` e una sezione "Scope di questa lezione" segnaposto). Preserva frontmatter e lesson-meta (aggiornando il badge da `bozza` allo stato indicato sopra), riscrivi la lesson-lead se la pagina lo richiede, e **sostituisci tutto il corpo** dopo la lead.

### Passo 0 — LEGGI PRIMA DI SCRIVERE (non saltare)

In ordine, leggi questi file per intero:

1. `AGENTS.md` — è il contratto di scrittura. Voce, profondità, struttura, regole sui termini, badge, anti-pattern. Quello che dice è vincolante.
2. `ai/foundations/come-funziona-un-llm.md` (lezione 0.1) — **gold standard di densità**. La tua lezione deve avere questa voce, questo ritmo, questa profondità. La tabella "Cosa un LLM non è" e la sezione "Sotto il cofano: la softmax" sono i riferimenti di stile.
3. `ai/foundations/embedding.md` (lezione 0.2) — **pavimento accettabile**. Stesso registro. Nota come usa `<details>` per sotto-il-cofano del coseno e per "come impara un modello di embedding".
4. `ai/SYLLABUS.md`, lettura mirata: il blocco della lezione corrente e i blocchi delle lezioni nominate nei `CROSS-REFERENCE` e nei bridge.

Quando hai finito, hai in testa: il timbro di voce, la densità attesa, la tabella anti-pattern come elemento obbligatorio, l'uso di `<details>` per il sotto-cofano matematico, il limite di non introdurre concetti che appartengono a lezioni successive.

### Template della pagina (da AGENTS.md §3, in ordine)

Usa i 13 blocchi del template di AGENTS.md §3. Promemoria critici:

1. **NESSUN emoji nei titoli di sezione.** Prosa pulita.
2. **Idea in una frase + ancora mentale**: il cuore della lezione in una riga, integrata nel discorso, non in un box separato.
3. **Una sezione H2 per ogni mattone.** Vedi i `CANDIDATI PER I MATTONI` sopra (sono un punto di partenza, non un vincolo).
4. **Sotto il cofano**: vedi blocco `SOTTO IL COFANO` sopra. Formula corretta con termini esatti **prima**, traduzione intuitiva **dopo**. Mai "salta la matematica": il lettore deve capirla, non evitarla.
5. **Tabella "Cosa NON è" — OBBLIGATORIA**: 4 righe, misconception comuni demolite. È un elemento gold-standard, non opzionale. Vedi `TABELLA "COSA NON È"` sopra per i candidati.
6. **Verifica di comprensione**: 6-7 domande a memoria. Le ultime 1-2 possono anticipare lezioni future. Stile identico a 0.1/0.2: invito allo spacing ("rivedi domani le incerte"), non meta-istruzioni nel corpo.
7. **Glossario** in fondo: solo i termini di **questa** pagina, definizioni di una riga, in ordine sensato (concettuale, non alfabetico forzato).
8. **Per approfondire**: 3-4 risorse. **MAI inventare URL, titoli precisi, numeri di pagina, nomi di paper specifici.** Indica cosa cercare e dove (es. "3Blue1Brown — serie su reti neurali, su YouTube"). Se sei certissimo di un URL, mettilo; in dubbio, descrivilo.
9. **Prossima lezione**: gancio in prosa breve verso la lezione indicata in `BRIDGE IN CHIUSURA`. Nome e numero esatti come nel SYLLABUS.

### Regole di voce (distillate da AGENTS.md §5 — vincolanti)

- **Italiano**. Diretto, va al sodo. Frasi brevi che si guadagnano il punto: o aprono una tensione o chiudono un colpo.
- **Tecnico vero**: usa i termini corretti, non annacquarli. Spiega ogni acronimo/termine alla **prima occorrenza, inline**, non in box separati né in un glossario a inizio pagina (le definizioni fuori contesto non si fissano).
- **Grassetti funzionali**: sul perno del concetto in quella frase, come appigli per l'occhio. Dosati. Se è tutto grassetto, niente lo è.
- **Battute dosate**: un guizzo ogni tanto, solo dove il concetto lo regge. NON a ogni paragrafo: stanca.
- **Analogie integrate nel discorso**, non in box separati. Devono affiancare la precisione, non sostituirla. Un'analogia semplifica e a volte "mente" un po': non farla passare per la verità tecnica completa.
- **Profondità ALTA**. Espandi finché il concetto è limpido, poi **fermati**. Chiarezza > lunghezza > brevità. Mai allungare per allungare.
- **Niente meta-istruzioni nel corpo** ("dillo a voce", "questo fissa il concetto"). Quelle vivono nei box `<details>`/quote e nella verifica finale.
- **Concreto, sempre**: termine astratto → esempio vivo la prima volta. Mai astrazione nuda.
- **Si fida del lettore**: non infantilizzare, non spiegare troppo l'ovvio, non ripetere lo stesso concetto in due paragrafi consecutivi.

### Termini con prerequisiti

Per concetti che presuppongono altre lezioni (es. "rete neurale", "transformer", "backpropagation", "tokenizer"): nomina con il termine corretto + una riga inline autosufficiente, e se serve un `<details>` di approfondimento minimo. **Non spiegarli da zero** se appartengono a un'altra lezione: rimanda. NON introdurre nel `<details>` ulteriori termini che richiederebbero a loro volta una spiegazione.

### Cross-reference

Usa **solo** i riferimenti elencati nel blocco `CROSS-REFERENCE VERIFICATI` sopra. Niente link a pagine inesistenti (es. `/radar`), niente numeri di lezione inventati, niente percorsi assoluti a caso. Quando linki, usa il path Docusaurus (es. `/foundations/embedding`) o il riferimento in prosa con numero esatto (es. "lezione 1.1").

### Output

Sovrascrivi il corpo del file indicato in `FILE`. Mantieni frontmatter e `<div class="lesson-meta">` (aggiornando il badge da `bozza` allo stato indicato in `BADGE DI STATO`, e la durata stimata). Riscrivi la `<p class="lesson-lead">` se necessario. **Non creare altri file.** **Non aggiungere commenti meta** tipo "ecco la lezione, ho seguito X regole". Consegna solo il file scritto.

### Checklist di auto-valutazione (eseguila prima di consegnare)

Prima di dichiarare finito, verifica TU stesso:

- [ ] Lette per intero `AGENTS.md`, 0.1, 0.2, e i blocchi SYLLABUS rilevanti.
- [ ] Apertura aggancia un filo della lezione precedente, non parte fredda.
- [ ] L'**idea in una frase** è presente e cristallina.
- [ ] C'è la tabella "Cosa NON è" con 4 righe non banali.
- [ ] C'è un `<details>` o un H3 "sotto il cofano" se la lezione ha una formula/algoritmo (vedi `SOTTO IL COFANO`).
- [ ] Nessun emoji nei titoli H2/H3.
- [ ] Nessun URL inventato, nessun titolo di paper/video di cui non sei certo.
- [ ] Bridge in chiusura punta alla lezione indicata, con nome e numero esatti dal SYLLABUS.
- [ ] Nessun concetto introdotto che appartiene a una lezione successiva (se ti serve nominarlo, rimanda).
- [ ] Densità confrontabile con la 0.1. Se rileggendo ogni paragrafo si guadagna il posto, sei a posto. Se hai allungato per allungato, taglia.
- [ ] La pagina compila in Docusaurus: frontmatter valido, MDX corretto, KaTeX wrappato in `$...$` o `$$...$$`, attributi HTML scritti come JSX-friendly (es. `class="..."` va bene in MDX 1 ma per sicurezza il progetto usa già `class` come negli altri file).
- [ ] Glossario in fondo, non a inizio pagina.
- [ ] Stato badge aggiornato (da `bozza` a quello indicato).

Quando hai dubbi su un dettaglio non coperto qui, scegli come avrebbero scelto 0.1 e 0.2.

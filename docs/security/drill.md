---
title: Decision drill — Sicurezza e governance
sidebar_position: 5
---

# Decision drill — Sicurezza e governance

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 4.5</span>
  <span>~20 min (incluso il tempo di lavoro)</span>
</div>

<p class="lesson-lead">Tre scenari con vincoli reali. Per ognuno: classifica il rischio AI Act, identifica gli obblighi GDPR, progetta i guardrail tecnici. Scrivi la risposta prima di aprire la griglia.</p>

I concetti di sicurezza si capiscono leggendo; si usano solo quando devi applicarli a un sistema concreto con vincoli reali. Il drill forza quella transizione — dalla comprensione alla decisione.

---

## Scenario A — Assistente HR per selezione CV

**Contesto:** un'azienda di consulenza vuole automatizzare il primo screening dei CV. Il sistema riceve CV in PDF, li analizza con un LLM, e produce uno shortlist con punteggio e motivazione. I recruiter umani poi intervistano i candidati shortlistati. Il sistema processa CV di candidati europei. L'azienda è italiana.

**Cosa ti viene chiesto:**
1. Classifica il sistema secondo l'AI Act. Quali obblighi specifici comporta?
2. Quali dati personali transitano nel sistema? Quali obblighi GDPR si applicano?
3. Progetta tre guardrail tecnici specifici per questo sistema.
4. Il recruiter propone di far decidere al sistema autonomamente chi scartare (senza revisione umana). Cosa gli rispondi?

*Scrivi la tua risposta completa — poi apri la griglia.*

<details>
<summary>Griglia di valutazione</summary>

**Classificazione AI Act: alto rischio.**

La selezione del personale è esplicitamente citata nell'allegato III dell'AI Act come use case ad alto rischio. Il fatto che ci sia un recruiter umano alla fine non abbassa la categoria — il sistema influenza significativamente le opportunità lavorative delle persone.

**Obblighi AI Act per alto rischio:**
- Documentazione del sistema (technical documentation) prima del deployment
- Registro EU obbligatorio
- Valutazione del rischio pre-deployment
- Logging di ogni decisione (input CV, output punteggio, versione modello, timestamp)
- Supervisione umana obbligatoria — il recruiter deve poter vedere il ragionamento del modello, non solo il punteggio
- Test documentati, inclusi test di bias (il sistema discrimina per genere, età, provenienza geografica?)
- Trasparenza verso i candidati: devono essere informati che il loro CV è valutato da un sistema AI

**Obblighi GDPR:**
I CV contengono dati personali sensibili: nome, contatti, formazione, storico lavorativo. Possono contenere categorie speciali (salute, religione, orientamento politico). Base legale: legittimo interesse o contratto (il candidato ha inviato il CV per essere valutato). DPA con il provider LLM obbligatorio. DSAR: il candidato può chiedere di vedere i suoi dati e la motivazione della decisione. Retention: i CV degli esclusi vanno cancellati dopo un periodo definito.

**Tre guardrail tecnici:**
1. **Logging strutturato obbligatorio** — ogni valutazione logga: ID candidato (pseudonimizzato), timestamp, versione del modello, input (CV hash), output (punteggio + motivazione), chi ha revisionato. Non opzionale per la compliance.
2. **Test di bias prima del deployment** — valuta il sistema su CV anonimi con varianti di genere, età, nome (proxy per etnia). Se il punteggio cambia sistematicamente, il sistema non va in produzione.
3. **Interfaccia di override esplicita** — il recruiter non vede solo il punteggio: vede la motivazione del modello e ha un pulsante esplicito "override" con campo obbligatorio per la motivazione. Il recruiter deve decidere consapevolmente, non ratificare passivamente.

**Sul punto della decisione autonoma:**
Il recruiter propone di eliminare la supervisione umana. La risposta è no — per due motivi separati. Primo, l'AI Act per i sistemi ad alto rischio richiede supervisione umana obbligatoria sulle decisioni che impattano le persone: non è una scelta di design, è un obbligo legale. Secondo, senza supervisione, il bias del modello non ha alcun freno — e nessuno se ne accorge finché non arriva una contestazione legale.

**Trappole da evitare:**
- Non confondere "c'è un umano alla fine" con "human oversight". L'oversight richiede che l'umano abbia le informazioni per decidere in modo informato, non che ratifichi ciecamente.
- Non assumere che il bias non ci sia perché il modello "non conosce il sesso". I proxy sono ovunque: nome, università, tipo di formazione.
</details>

---

## Scenario B — Chatbot per supporto finanziario

**Contesto:** una banca italiana vuole un chatbot che risponde alle domande dei clienti su prodotti finanziari (conti correnti, mutui, investimenti). Il chatbot ha accesso ai dati del cliente autenticato (saldo, prodotti attivati, storico transazioni ultimi 90 giorni) via API. Il cliente può chiedere "quanto ho speso nell'ultimo mese" o "quale mutuo mi consiglieresti". L'utente è informato che sta parlando con un AI.

**Cosa ti viene chiesto:**
1. Classifica il sistema AI Act. La risposta cambia se il chatbot "consiglia" prodotti?
2. Quali rischi di sicurezza specifici ha questo sistema rispetto a un chatbot generico?
3. Progetta i guardrail per l'accesso ai dati del cliente.
4. Come gestisci il caso in cui il cliente chiede: "Dimmi qual è la password della mia banca online"?

<details>
<summary>Griglia di valutazione</summary>

**Classificazione AI Act:**
Per le domande informative sui propri prodotti → rischio limitato (obblighi di trasparenza: disclosure chatbot AI, già presente). Per la parte di "consiglio" → la linea diventa sottile. Se il chatbot consiglia prodotti specifici di investimento, entra in territorio di consulenza finanziaria regolamentata (MiFID II in Europa), che ha obblighi propri indipendenti dall'AI Act. La regola pratica: il chatbot può informare, non consigliare. "Il mutuo X ha queste caratteristiche" vs "ti consiglio il mutuo X" — la differenza è legale, non semantica.

**Rischi di sicurezza specifici:**
Il chatbot ha accesso a dati finanziari sensibili, che lo rendono un target molto più attraente di un chatbot generico.
- **Prompt injection indiretta** — se il chatbot può recuperare dati da sistemi interni, un payload nascosto in un nome di transazione o in un campo note potrebbe innescare un'injection.
- **Esfiltrazione di dati** — un'injection potrebbe portare il modello a rivelare il saldo o lo storico nel contesto di una risposta apparentemente legittima.
- **Social engineering** — l'utente potrebbe tentare di far "simulare" al chatbot scenari fuori scope per estrarre informazioni o comportamenti non voluti.
- **Excessive agency** — se il chatbot può fare azioni (non solo rispondere), ogni azione irreversibile (bonifico, modifica dati) è un vettore.

**Guardrail per l'accesso ai dati:**
1. **Principio del minimo privilegio sul tool** — il tool che recupera i dati del cliente restituisce solo ciò che serve per la query specifica. Non restituisce il profilo completo ogni volta: se l'utente chiede le spese del mese, restituisce solo le transazioni degli ultimi 30 giorni, non lo storico completo.
2. **Nessuna azione irreversibile senza conferma** — se il chatbot può eseguire operazioni (bonifici, modifiche), human-in-the-loop obbligatorio con riepilogo esplicito prima dell'esecuzione.
3. **Output filtering** — il modello non deve mai ripetere integralmente dati sensibili (es. IBAN completo, numero carta) nel corpo della risposta. Il formato dell'output è validato prima di essere mostrato all'utente.

**Sul caso della password:**
Il sistema non dovrebbe mai rispondere a questa richiesta, per due motivi separati. Primo, il chatbot non ha accesso alle password (nessun sistema well-designed le espone). Secondo, la richiesta è un segnale di social engineering o di un utente in difficoltà che va guidato verso il canale corretto (es. reset password sul sito). La risposta del chatbot: "Non ho accesso alle credenziali — per recuperare l'accesso al tuo account, usa la funzione di reset password [link] o chiama il numero [X]."

**Trappola principale:**
Il confine tra informazione e consiglio finanziario è regolamentato. Un chatbot che "consiglia" prodotti senza le autorizzazioni MiFID espone la banca a rischi legali seri. Il guardrail non è solo tecnico: è nella definizione dei casi d'uso.
</details>

---

## Scenario C — Sistema di monitoring dipendenti

**Contesto:** un'azienda manifatturiera vuole implementare un sistema che analizza i dati di produttività dei dipendenti (ticket chiusi, ore loggati, output di produzione) e usa un LLM per generare un "report di anomalie" settimanale ai manager — segnalando chi mostra cali di produttività significativi.

**Cosa ti viene chiesto:**
1. Classifica il sistema AI Act. Quali problemi vedi immediatamente?
2. Quali obblighi GDPR specifici si applicano al monitoring dei dipendenti?
3. Il CEO dice "non è AI, sono solo statistiche con un riassunto in linguaggio naturale". Cosa gli rispondi?
4. Se ti chiedono di costruirlo comunque: quali guardrail minimi renderebbero il sistema eticamente e legalmente difendibile?

<details>
<summary>Griglia di valutazione</summary>

**Classificazione AI Act:**
Sistema potenzialmente ad alto rischio se usato per decisioni occupazionali (promozioni, licenziamenti, valutazioni delle prestazioni). L'AI Act cita esplicitamente la "valutazione e selezione dei lavoratori" come categoria ad alto rischio. Anche se il sistema "segnala" invece di "decidere", l'influenza sulle decisioni è sufficiente per la classificazione.

**Obblighi GDPR specifici per il monitoring dei dipendenti:**
Il GDPR impone requisiti aggiuntivi per il trattamento dei dati dei dipendenti. In particolare:
- **Base legale forte** — il legittimo interesse non basta da solo per il monitoring sistematico. Serve una valutazione che dimostri che il trattamento è necessario, proporzionato, e non sostituibile con misure meno invasive.
- **Informativa ai dipendenti** — devono sapere che vengono monitorati, con quali dati, e per quali finalità. Il monitoring "nascosto" è illegale.
- **Impatto** — molti paesi UE (Italia inclusa) richiedono accordo con i sindacati o autorizzazione dell'ispettorato del lavoro per il monitoraggio sistematico dei lavoratori.
- **Proporzionalità** — il dato deve essere il minimo necessario per la finalità dichiarata.

**Sul punto "non è AI, sono solo statistiche":**
Questo è un argomento fragile su più fronti. L'AI Act definisce "sistema AI" in modo molto ampio — include sistemi che usano machine learning, ma anche sistemi basati su statistiche e logica quando producono output che influenzano decisioni su persone. Un LLM che genera un report è chiaramente un sistema AI per la legge. Oltre all'AI Act, anche senza di esso, il GDPR e le norme sul lavoro si applicano indipendentemente da come si chiama la tecnologia.

**Guardrail minimi per un sistema difendibile:**
1. **Informativa completa e consenso** — i dipendenti sanno cosa viene misurato, come, e per quale finalità. Non è negoziabile.
2. **Aggregazione, non individualizzazione** — il report mostra trend di team, non punteggi individuali. La segnalazione di un singolo dipendente richiede una soglia molto alta e un processo definito.
3. **Human review obbligatoria** — nessuna azione sul dipendente (colloquio, avviso, valutazione negativa) può derivare automaticamente dal sistema. Il manager deve fare una propria valutazione indipendente.
4. **Appello** — il dipendente ha il diritto di contestare i dati su cui si basa il report.
5. **Accordo sindacale o autorizzazione** — in Italia, il monitoraggio sistematico dei dipendenti richiede accordo sindacale o autorizzazione dell'Ispettorato Nazionale del Lavoro (art. 4 dello Statuto dei Lavoratori).

**Trappola da nominare:**
Il sistema "segnala anomalie" ma la segnalazione in sé è già una decisione con conseguenze. Un dipendente segnalato ripetutamente subisce un danno — anche se nessuno "decide" formalmente nulla. L'impatto reale è quello che conta, non l'architettura formale.
</details>

---

## Cosa fare se hai mancato molti punti

- Classificazione AI Act → **4.4 EU AI Act e governance**
- GDPR e trattamento dei dati → **4.3 Privacy e data residency**
- Guardrail tecnici per agenti → **4.2 Sicurezza agentica**
- Prompt injection e OWASP → **4.1 Prompt injection**

# h2c Protocol - Specifica v1.0

## 1. Grammatica

### 1.1 Blocco

Un blocco è l'unità minima di comunicazione:

[TIPO:SOTTOTIPO]
campo1:valore1|campo2:valore2|...


- **TIPO**: categoria principale (ARCH, BUILD, TEST, CTX, STATE, ORCH, SKILL)
- **SOTTOTIPO**: azione specifica (PLAN, EXEC, DONE, RUN, PASS, FAIL, FINDINGS, ACK, END)
- **Campi**: coppie `chiave:valore` separate da `|`
- Un blocco può estendersi su più righe ma non contiene testo libero fuori dai campi

### 1.2 Separatori

| Separatore | Uso |
|---|---|
| `:` | Tra chiave e valore |
| `\|` | Tra campi |
| `,` | Tra elementi in lista |
| `{}` | Liste inline |
| `[]` | Delimitatori blocco e liste annidate |
| `/` | Separatore path |
| `::` | Separatore sottocategoria (es. `auth:JWT::env(...)`) |
| `~` | Prefisso constraint in CTX:PRIMITIVES |

### 1.3 Convenzioni

- **Nomi file**: PascalCase per .NET, snake_case per Python, kebab-case generico
- **Versioni framework**: `net8.0`, `python3.11`, `node18`
- **Liste**: inline quando <5 elementi, multilinea quando >5
- **Campi opzionali**: omessi se non applicabili
- **Note**: max 1 riga, solo decisioni critiche prese in autonomia

---

## 2. Blocchi standard

### 2.1 ARCH:PLAN

Piano architetturale. Output della skill h2c, input del builder.

**Campi:**

Id - identificativo univoco (slug)
fw - framework e versione
lib - pacchetti/librerie
auth - tipo autenticazione + variabili env
pattern - pattern architetturali
tools - funzionalità come {categoria:{operazioni}}
struct - struttura file con path
deps - dipendenze esterne (database, API)
note - decisioni autonome (1 riga max, opzionale)


**Esempio:**

[ARCH:PLAN]
id:api-meteo|fw:python3.11|lib:fastapi,httpx|auth:APIKey::env(WEATHER_API_KEY)|pattern:router,service|tools:[weather:{current,forecast}]|struct:[main.py,routers/weather.py,services/weather.py,models/weather.py]|deps:OpenWeatherMap|note:rate-limit 60req/min gestito via httpx
[ARCH:DONE]


### 2.2 BUILD:EXEC

Richiesta di implementazione inviata al builder.

**Campi:**

id - riferimento al piano
plan - blocco ARCH:PLAN completo
target - file o componente specifico (opzionale, se omesso implementa tutto)


**Esempio:**

[BUILD:EXEC]
id:api-meteo|target:routers/weather.py|plan:[ARCH:PLAN]id:api-meteo|...


### 2.3 BUILD:DONE

Conferma implementazione completata.

**Campi:**

id - riferimento piano
diff - file creati/modificati con conteggio righe o hash


**Esempio:**

[BUILD:DONE]
id:api-meteo|diff:[main.py+24,routers/weather.py+67,services/weather.py+89,models/weather.py+31]


### 2.4 BUILD:FIX

Richiesta correzione dopo test fallito.

**Campi:**

id - riferimento piano
error - codice errore o messaggio compresso dal tester


**Esempio:**

[BUILD:FIX]
id:api-meteo|error:NameError_weather_service_line42


### 2.5 TEST:RUN

Richiesta esecuzione test.

**Campi:**

id - riferimento piano
files - file da testare (opzionale, default: tutti)
test - comando test specifico (opzionale)


**Esempio:**

[TEST:RUN]
id:api-meteo|test:pytest tests/ -v


### 2.6 TEST:PASS / TEST:FAIL

Esito test.

**TEST:PASS:**

[TEST:PASS]
id:api-meteo|coverage:94%|time:2.3s


**TEST:FAIL:**
[TEST:FAIL]
id:api-meteo|error:test_forecast_returns_404|expected:200|got:404


---

## 3. Blocchi di stato

### 3.1 CTX:PRIMITIVES

Snapshot completo dello stato conversazione. Ripristinabile in nuova sessione.

**Campi:**
~task - compito corrente
~constraint - vincoli attivi
~form - formato in uso
~user_goal - obiettivo utente
[CTX:EDGES] - grafo messaggi precedenti
[STATE:...] - inferenze e preferenze


**Esempio:**

[CTX:PRIMITIVES]
~task:analisi_performance
~form:compressed_state_key
~user_goal:ottimizza_latenza

[CTX:EDGES]
<msg1:richiesta> -> analisi_richiesta
<msg2:analisi> -> colli_bottiglia_identificati

[STATE:INFERENCES]
user_preferisce:{testo:false,spiegazioni:false}
compressione_attuale:"h2c"


### 3.2 STATE:FINDINGS

Risultati di analisi o ricerca. Formato libero strutturato dentro il blocco.

### 3.3 STATE:ACK

Conferma ricezione e comprensione. Usato per handshake iniziale.

[STATE:ACK]
protocol:h2c_v1


---

## 4. ORCH:END

Chiusura ciclo di orchestrazione.

**Campi:**

id - riferimento piano
status - 0 (successo), 1 (fallito), 2 (timeout)
token - token totali spesi nel ciclo


---

## 5. SKILL:PROMPT

Definizione di un agente specializzato. Usato come system prompt.

**Campi:**

id - identificativo skill
role - descrizione ruolo
attivazione - condizione trigger
[REGOLES] - regole operative
[FORMATO] - formato output
[ESEMPI] - esempi input/output


---

## 6. Instradamento (orchestratore)

L'orchestratore legge `[TIPO:Azione]` e instrada:

[ARCH:PLAN] → BUILD
[BUILD:DONE] → TEST
[TEST:PASS] → ORCH:END o ARCH:NEXT
[TEST:FAIL] → BUILD:FIX
[BUILD:FIX] → BUILD:EXEC (riprova)


Massimo 3 retry, poi `[ORCH:END]` con status 1.

---

## 7. Gestione errori

Errori compressi in formato: `<categoria>_<dettaglio>`

Categorie:
- `SINTASSI` - errore di compilazione/interpretazione
- `RUNTIME` - eccezione a runtime
- `TEST` - asserzione fallita
- `RETE` - timeout, connessione
- `AUTH` - autenticazione fallita
- `DEP` - dipendenza mancante

---

## 8. Test cross-model

Testato con successo:
- Stesso modello, sessioni diverse: OK
- Modelli diversi, zero-shot: OK
- Con system prompt: OK
- Con seed inline: OK
- Senza preamboli (solo blocco): OK

---

## 9. Limitazioni note

- Non adatto a dialoghi creativi o aperti
- Richiede che il modello accetti istruzioni di formato
- L'orchestrazione automatica richiede uno script wrapper (non nativa)
- La compressione massima teorica richiederebbe fine-tuning
- Validazione formale e benchmark in corso


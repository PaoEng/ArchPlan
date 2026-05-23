# ArchPlan

## Protocollo per dialogo AI-to-AI a token minimi

Obiettivo: ridurre i token mantenendo chiarezza, stato condiviso e sicurezza.
EN: minimal-token AI-to-AI protocol with fixed fields and compact payloads.

### Regole base

1. **Una riga = un atto**.
2. Ogni messaggio usa struttura fissa con delimitatore `|`: `T|id|k|payload`
   - `id` = intero progressivo univoco per conversazione
   - al raggiungimento di `2147483647`, riavvio da `1` annunciato con `S|rollover=y`; nessun messaggio in-flight durante il cambio
3. `k` (kind) consentiti:
   - `Q` = domanda
   - `A` = risposta
   - `C` = comando operativo
   - `S` = stato/sintesi
   - `E` = errore
4. `payload` è compatto:
   - chiavi corte (`obj`, `res`, `err`, `next`), elenco non esaustivo
   - significati base: `obj`=obiettivo, `res`=risultato, `err`=errore, `next`=prossimo passo
   - nuove chiavi in `snake_case` breve
   - nuove chiavi vanno documentate nel primo messaggio `S` del task (`keydef=<chiave>:<significato>` con `:` interno codificato)
   - valori enumerati quando possibile
   - caratteri riservati in percent-encoding: spazio=`%20`, `;`=`%3B`, `|`=`%7C`, `=`=`%3D`, `:`=`%3A`, `%`=`%25`, newline=`%0A`, carriage return=`%0D`, tab=`%09`
   - caratteri non riservati RFC3986 (`ALPHA`, `DIGIT`, `-`, `.`, `_`, `~`) restano in chiaro
   - nessuna prosa superflua
5. Ogni turno include al massimo:
   - 1 obiettivo
   - 1 risultato
   - 1 prossimo passo

### Formato

```text
T|<id>|<k>|k1=v1;k2=v2
```

Esempi:

```text
T|17|Q|obj=test_scope;ctx=api_auth
T|18|A|res=2_fail;err=timeout;next=retry_1
T|19|C|act=run;cmd=pytest%20-k%20auth
T|20|S|res=pass;next=close
```

### Compressione semantica consigliata

- usare codici stabili (`pass/fail`, `hi/med/low`, `y/n`)
- evitare ripetizioni: riferirsi a `id` precedente
- inviare dettagli estesi solo su richiesta esplicita (`T|<id>|Q|obj=detail`)

### Regola di fallback

Se il messaggio è ambiguo, inviare:

```text
T|<id>|E|err=ambiguous;next=clarify
```
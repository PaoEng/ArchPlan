# ArchPlan

## Protocollo per dialogo AI-to-AI a token minimi

Obiettivo: ridurre i token mantenendo chiarezza, stato condiviso e sicurezza.

### Regole base

1. **Una riga = un atto**.
2. Ogni messaggio usa prefisso fisso: `T|ID|K|PAYLOAD`
3. `K` (kind) consentiti:
   - `Q` = domanda
   - `A` = risposta
   - `C` = comando operativo
   - `S` = stato/sintesi
   - `E` = errore
4. `PAYLOAD` è compatto:
   - chiavi corte (`obj`, `res`, `err`, `next`)
   - valori enumerati quando possibile
   - nessuna prosa superflua
5. Ogni turno include al massimo:
   - 1 obiettivo
   - 1 risultato
   - 1 prossimo passo

### Formato

```text
T|<id>|<K>|k1=v1;k2=v2
```

Esempi:

```text
T|17|Q|obj=test_scope;ctx=api_auth
T|18|A|res=2_fail;err=timeout,next=retry_1
T|19|C|act=run;cmd=pytest_-k_auth
T|20|S|res=pass;next=close
```

### Compressione semantica consigliata

- usare codici stabili (`pass/fail`, `hi/med/low`, `y/n`)
- evitare ripetizioni: riferirsi a `id` precedente
- inviare dettagli estesi solo su richiesta esplicita (`Q detail`)

### Regola di fallback

Se il messaggio è ambiguo, inviare:

```text
T|<id>|E|err=ambiguous;next=clarify
```
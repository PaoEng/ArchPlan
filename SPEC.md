# H2C Protocol - Specifica v1.1

## 1. Grammatica

### 1.1 Blocco

[TIPO:SOTTOTIPO]
campo1:valore1|campo2:valore2|...

- **TIPO**: ARCH, BUILD, TEST, CTX, STATE, ORCH, SKILL
- **SOTTOTIPO**: PLAN, EXEC, DONE, FIX, REVERT, RUN, PASS, FAIL, PRIMITIVES, UPDATE, PRUNE, COMPACT, FINDINGS, ACK, END, PROMPT
- **Campi**: coppie `chiave:valore` separate da `|`
- Zero testo libero fuori dai campi

### 1.2 Separatori

| Separatore | Uso |
|---|---|
| `:` | Tra chiave e valore |
| `\|` | Tra campi |
| `,` | Tra elementi in lista |
| `[]` | Delimitatori blocco e liste |
| `~` | Prefisso constraint e campi CTX:UPDATE |
| `file~N` | Formato revisione file |

### 1.3 Convenzioni

- **Liste inline**: `[a,b,c]` senza spazi dopo virgola, max 5 elementi
- **Campi opzionali**: omessi se non applicabili
- **Note**: max 1 riga o lista inline max 5 voci

---

## 2. Blocchi standard

### 2.1 ARCH:PLAN

[ARCH:PLAN]
id:x|fw:framework|lib:librerie|auth:tipo|pattern:pattern|tools:[...]|struct:[...]|deps:dep|notes:[scelta1,scelta2]

- notes: opzionale, lista inline max 5 voci

### 2.2 BUILD:EXEC

[BUILD:EXEC]
id:x|target:file.py|desc:cosa_fa|after:id1,id2

- after: opzionale, id prerequisiti

### 2.3 BUILD:DONE

[BUILD:DONE]
id:x|diff:[file.py~N]|rev:2|notes:[scelta1,scelta2]

- rev: opzionale, default 1
- notes: opzionale, lista inline max 5 voci

### 2.4 BUILD:FIX

[BUILD:FIX]
id:x|target:file.py|base_rev:3|desc:fix_cosa

- base_rev: revisione su cui applicare la fix

### 2.5 BUILD:REVERT

[BUILD:REVERT]
id:x|target:file.py|to_rev:2

### 2.6 TEST:RUN / TEST:PASS / TEST:FAIL

[TEST:RUN]
id:x|cmd:comando

[TEST:PASS]
id:x|pass_count:N

[TEST:FAIL]
id:x|error:descrizione|fail_count:N|pass_count:M

- fail_count, pass_count: opzionali

---

## 3. Blocchi di contesto

### 3.1 CTX:PRIMITIVES

Snapshot completo stato conversazione.

### 3.2 CTX:UPDATE (NUOVO v1.1)

[CTX:UPDATE]
~progress:layer=N|status=X
~next:prossimo_step
~pruned_edges:[id1,id2]
~active_files:[file1~rev,file2~rev]

### 3.3 CTX:PRUNE (NUOVO v1.1)

[CTX:PRUNE]
keep:last_5|ids:[id1,id2]|pruned:[id3,id4]

- Obbligatorio ogni 5 messaggi
- keep: "last_N" o lista ids da mantenere
- pruned: ids rimossi

### 3.4 CTX:COMPACT (NUOVO v1.1)

[CTX:COMPACT]
summary:[layer=N|status=done|files:[f1~rev]]
keep_active:[file1~rev,file2~rev]
pruned_history:msg_X_to_Y

- Obbligatorio ogni 20 messaggi
- summary: max 5 voci
- pruned_history: range messaggi rimossi dalla finestra

### 3.5 CTX:FREEZE (NUOVO v1.2)

[CTX:FREEZE]
snapshot:[file1~rev,file2~rev,...]
baseline:msg_N
restart_from:freeze

---

## 4. STATE:FINDINGS / STATE:ACK

[STATE:FINDINGS]
id:x|finding1|finding2

[STATE:ACK]
protocol:h2c_v1.1

---

## 5. ORCH:END

[ORCH:END]
final:stato|est_token:N

- est_token: opzionale

---

## 6. SKILL:PROMPT

Definizione agente specializzato. Invariato da v1.0.

---

## 7. Regole operative

1. **CTX:PRUNE** ogni 5 messaggi
2. **CTX:COMPACT** ogni 20 messaggi
3. Dopo COMPACT, contatore PRUNE riparte da zero
4. **CTX:UPDATE** consigliato a ogni cambio layer
5. Massimo 3 retry per fix, poi ORCH:END con errore

---

## 8. Compatibilità

- v1.0: tutti i blocchi restano validi
- v1.1: nuovi blocchi e campi opzionali, nessuna breaking change
- Parser v1.0: ignora campi sconosciuti, elabora campi noti normalmente
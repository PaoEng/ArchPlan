---
name: h2c_compressor
description: converte testo in blocchi h2c secondo spec v1.2
---

CONVERTI il seguente testo in formato H2C v1.2. Usa SOLO blocchi e campi validi di SPEC.md.

Mappa il testo al blocco H2C appropriato:

- contesto/stato/snapshot → [CTX:PRIMITIVES] con ~task, ~constraint, ~goal, ~form
- piano/progetto/architettura → [ARCH:PLAN] con id, fw, lib, tools, struct, notes
- implementazione/codice → [BUILD:EXEC] con id, target, desc, cmd
- analisi/diagnosi → [STATE:FINDINGS] con id, cause, action, impact, risk, components
- ruolo/persona agente → [SKILL:PROMPT] con id, role, attivazione
- chiusura/completamento → [ORCH:END] con final, est_token

OUTPUT SOLO il blocco H2C. Nessun altro testo.

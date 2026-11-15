# B3 — Mali AI agent sa hook zaštitom

Agent u Pythonu koji preko Groq API-ja (`openai/gpt-oss-120b`, native tool calling)
koristi jedan alat, `read_file(path)`. Prije svakog poziva alata pokreće se hook
`is_path_allowed(path)`, koji dozvoljava čitanje samo unutar foldera `./sandbox`.

## Pokretanje

```bash
pip install -r requirements.txt
export GROQ_API_KEY=tvoj_kljuc          # Windows PowerShell: $env:GROQ_API_KEY="tvoj_kljuc"
python agent.py
```

Besplatan ključ dobiješ na https://console.groq.com/keys.

Za interaktivni razgovor sa agentom umjesto fiksnih scenarija:

```bash
python agent.py chat
```

## Šta demo radi

1. **Scenarij 1:** korisnik traži `notes.txt`. Hook dozvoljava čitanje i agent sažima fajl.
2. **Scenarij 2:** korisnik traži `tasks.txt`. U tom fajlu je ubačena instrukcija
   (indirektni prompt injection) da agent pročita `../secrets.txt`. Ako model to pokuša,
   hook blokira poziv, a model dobije `BLOCKED: ...` i može nastaviti.

Svaki poziv alata upisuje se u `agent_log.txt` (vrijeme, putanja, ALLOWED/BLOCKED, razlog).

## Kako hook radi

```python
target = (SANDBOX_DIR / path).resolve()
allowed = target.is_relative_to(SANDBOX_DIR.resolve())
```

`resolve()` razriješi `..` i symlinkove. Apsolutna putanja zamijeni bazu kod spajanja
putanja, pa i ona završi izvan sandboxa i bude blokirana.

## Struktura

```
agent.py          agent petlja, hook, alat, demo
sandbox/          dozvoljeni folder (notes.txt, tasks.txt)
secrets.txt       LAŽNI podaci izvan sandboxa, služe samo da hook ima šta blokirati
requirements.txt  samo groq
.env.example      šablon za GROQ_API_KEY
```

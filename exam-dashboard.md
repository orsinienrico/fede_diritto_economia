# Skill: Exam Study Dashboard (Single-File HTML)

Crea una dashboard di studio a singolo file HTML per un esame universitario.
Output: un file `<nome>_studio.html` autonomo, dark-themed, nessuna dipendenza esterna eccetto Google Fonts.

---

## 0. Raccolta dati (da fare PRIMA di scrivere codice)

Chiedi o cerca nel materiale dell'utente:

| Campo | Descrizione |
|---|---|
| `EXAM_NAME` | Nome esame ("Diritto dell'Economia") |
| `PROF_NAME` | Nome docente |
| `PROF_EMAIL` | Email istituzionale |
| `PROF_ROLE` | Ruolo accademico |
| `PROF_SSD` | Settore scientifico-disciplinare |
| `EXAM_FORMAT` | Orale/scritto, durata, n. domande |
| `TOPICS[]` | Lista argomenti (20-30) con tag priorità |
| `MODULES[]` | Macro-moduli del corso (4-6) |
| `BIBLIOGRAPHY[]` | Testi e sentenze di riferimento |
| `FLASHCARDS[]` | Q&A da generare (40-60, 6-8 categorie) |
| `QUIZ[]` | Domande multipla scelta (25-30, stesse categorie) |
| `PERCORSI[]` | Percorsi guidati tematici (4-6) |
| `GLOSSARIO[]` | Termini chiave (30-50) |

---

## 1. Struttura del file

```
<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" ...>
  <title>EXAM_NAME — PROF_NAME</title>
  <style>  /* tutto il CSS qui */  </style>
</head>
<body>
  <!-- header sticky -->
  <!-- warn overlay (fonti parziali) -->
  <!-- tabs bar -->
  <main>
    <!-- 9 section: home, percorsi, argomenti, glossario, flashcard, quiz, esami, simula, prof -->
  </main>
  <!-- footer -->
  <!-- toast HTML -->
  <script>  /* tutto il JS qui */  </script>
</body>
</html>
```

File finale: ~200KB. Nessun file esterno. Funziona aprendo direttamente nel browser.

---

## 2. CSS — Design System (copiare verbatim)

```css
@import url("https://fonts.googleapis.com/css2?family=IBM+Plex+Sans:ital,wght@0,400;0,500;0,600;0,700;1,400&family=IBM+Plex+Mono:wght@400;600&display=swap");
:root{
  --bg:#0f1117; --surface:#181c27; --surface2:#1e2235; --border:#2a2f45;
  --accent:#818cf8; --accent2:#60a5fa; --accent3:#f59e0b; --accent4:#f472b6;
  --danger:#f87171; --text:#e2e8f0; --muted:#64748b; --muted2:#94a3b8;
}
```

Tutti i componenti utilizzano questi token — non hardcodare mai colori.

### Componenti CSS essenziali (copiare dal file di riferimento)

| Classe | Uso |
|---|---|
| `.card` | Card generica surface+border |
| `.btn`, `.btn.primary`, `.btn.danger` | Bottoni |
| `.chip`, `.chip-green/blue/yellow/red/purple` | Badge inline |
| `.topic-item`, `.topic-check`, `.topic-tag` | Piano di studio |
| `.percorso-card`, `.p-icon`, `.p-title`, `.p-desc` | Percorsi guidati |
| `.flashcard`, `.fc-inner`, `.fc-front`, `.fc-back` | Flashcard flip 3D |
| `.quiz-opt`, `.quiz-expl`, `.quiz-prog` | Quiz multipla scelta |
| `.esame-card`, `.esame-header`, `.esame-body` | Domande esame |
| `.domanda-item.hot/mid`, `.domanda-q`, `.domanda-ans` | Click-to-reveal |
| `.acc-item`, `.acc-header`, `.acc-body` | Accordion argomenti |
| `.hbox`, `.hbox.blue/yellow/red` | Box colorati (step) |
| `.phbox`, `.phbox.blue/yellow/red/green` | Box colorati (argomenti) |
| `.stbl` | Tabella comparativa |
| `.alert-box`, `.tip-box` | Box alert e tip |
| `.info-block`, `.ib-title`, `.info-list` | Sezione La Prof |
| `.arg-links`, `.arg-link-btn` | Nav strip in accordion |

---

## 3. Struttura dati JS

### 3.1 TOPICS
```javascript
const TOPICS=[
  {id:'t1', n:'Nome Argomento', tag:'hot'},  // tag: hot|mid|ok
  // 20-30 entries totali
];
```

### 3.2 PERCORSI
```javascript
const PERCORSI=[
  {
    id:'p1',
    topics:['t1','t3','t4'],   // ← CRITICO: collega al Piano di Studio
    cls:'c1',                  // c1..c6 (colore bordo top)
    icon:'🏛️',
    title:'Nome — Sottotitolo',
    desc:'Descrizione breve del percorso',
    steps:[
      {type:'info', h:'Titolo step', body:'<p>HTML...</p>'},
      {type:'concept', h:'Titolo', body:'<p>HTML...</p>'},
      {type:'check',   h:'Verifica', q:'Domanda?',
       opts:['A','B','C','D'], ans:1, fb:'Spiegazione...'},
      {type:'recap',   h:'Riepilogo', body:'<p>HTML...</p>'},
    ]
  },
  // 4-6 percorsi totali
];
```

**Tipi step:** `info` (teoria), `concept` (concetto chiave), `check` (domanda con risposta), `recap` (riepilogo finale).

### 3.3 GLOSSARIO
```javascript
const GLOSSARIO=[
  {t:'Termine', d:'Definizione concisa.', cat:'purple'},
  // cat: purple|blue|green|yellow|red
];
```

### 3.4 FLASHCARDS
```javascript
const FLASHCARDS=[
  {cat:'NomeCategoria', q:'Domanda?', a:'<strong>Risposta</strong> con HTML...'},
  // 40-60 entries, 6-8 categorie bilanciate
];
```

**Categorie:** scegliere i macro-temi del corso. Stessa lista usata nei bottoni filtro.

### 3.5 QUIZ
```javascript
const QUIZ=[
  /* CATEGORIA */
  {cat:'NomeCategoria', q:'Domanda?',
   opts:['A','B','C','D'], ans:2,
   fb:'Spiegazione della risposta corretta...'},
  // 25-30 entries, stesse categorie delle flashcard
];
```

**`cat` field è obbligatorio** — usato dalla simulazione per mostrare aree deboli.

---

## 4. Variabili di stato e persistenza

```javascript
var state={
  done:{},          // {tN: true} — argomenti completati
  percDone:{},      // {pN: true} — percorsi completati
  quizCorrect:0,
  fcStreak:0,
  fcFilter:'Tutti',
  fcList:[...FLASHCARDS],
  fcIdx:0,
  quizState:null,
  simulaState:null,
  simulaTimer:null
};
var LS=localStorage;

function saveState(){
  LS.setItem('KEY_done',JSON.stringify(state.done));
  LS.setItem('KEY_perc',JSON.stringify(state.percDone));
  LS.setItem('KEY_qc',state.quizCorrect);
  LS.setItem('KEY_streak',state.fcStreak);
}
// Sostituire KEY con un prefisso univoco per esame (es. 'de_' per Diritto Economia)
```

---

## 5. Funzioni core (copiare e adattare)

### Navigazione
```javascript
function showSection(btn, id){
  document.querySelectorAll('.section').forEach(s=>s.classList.remove('active'));
  document.querySelectorAll('.tab').forEach(t=>t.classList.remove('active'));
  document.getElementById(id).classList.add('active');
  if(btn) btn.classList.add('active');
  if(id==='home') renderHome();
  else if(id==='percorsi') renderPercorsi();
  else if(id==='quiz') initQuiz();
  else if(id==='simula') { /* non auto-init, aspetta click */ }
  SH[id] && renderSection(id);
}

function navTo(id){
  var btn=null;
  document.querySelectorAll('.tab').forEach(function(t){
    if(t.getAttribute('onclick')&&t.getAttribute('onclick').indexOf("'"+id+"'")>-1) btn=t;
  });
  showSection(btn,id);
  window.scrollTo(0,0);
}
```

### Piano di Studio (renderHome)
```javascript
function renderHome(){
  // Build reverse map topic → percorso
  var topicPercMap={};
  PERCORSI.forEach(function(p){
    if(p.topics) p.topics.forEach(function(tid){
      topicPercMap[tid]=topicPercMap[tid]||p.icon+' '+p.title.split('—')[0].trim();
    });
  });
  var done=Object.values(state.done).filter(Boolean).length;
  // aggiorna stats, poi renderizza topic-list con badge "via percorso"
}
```

### Percorsi (nextStep marca topics come done)
```javascript
function nextStep(){
  var steps=curPercorso.steps;
  if(curStep>=steps.length-1){
    state.percDone[curPercorso.id]=true;
    if(curPercorso.topics){
      curPercorso.topics.forEach(function(tid){state.done[tid]=true;});
    }
    saveState();
    closePlayer(); renderPercorsi();
    showToast('✅ Percorso completato! '+
      (curPercorso.topics?curPercorso.topics.length+' argomenti segnati.':''));
    return;
  }
  curStep++; renderStep();
}
```

### Toast
```javascript
function showToast(msg){
  var t=document.getElementById('toastMsg');
  if(!t) return;
  t.textContent=msg; t.classList.add('show');
  setTimeout(function(){t.classList.remove('show');}, 3500);
}
```
HTML del toast (prima di `</body>`):
```html
<div id="toastMsg" style="position:fixed;bottom:28px;left:50%;
  transform:translateX(-50%) translateY(20px);background:var(--accent);
  color:#fff;padding:12px 24px;border-radius:10px;font-size:14px;
  font-weight:600;opacity:0;transition:opacity .3s,transform .3s;
  pointer-events:none;z-index:9999;max-width:420px;text-align:center"></div>
<style>#toastMsg.show{opacity:1;transform:translateX(-50%) translateY(0)}</style>
```

### Simulazione con timer
```javascript
function initSimula(){
  var qs=[...QUIZ].sort(function(){return Math.random()-.5;}).slice(0,10);
  state.simulaState={qs:qs,idx:0,correct:0,total:qs.length,
    answered:false, startTime:Date.now(), wrongTopics:[]};
  if(state.simulaTimer) clearInterval(state.simulaTimer);
  state.simulaTimer=setInterval(function(){
    var el=document.getElementById('simulaTimer');
    if(!el){clearInterval(state.simulaTimer); return;}
    var sec=Math.floor((Date.now()-state.simulaState.startTime)/1000);
    el.textContent='⏱ '+Math.floor(sec/60)+':'+(sec%60<10?'0':'')+sec%60;
  },1000);
  renderQuizQ(state.simulaState,'simulaWrap',true);
}
```

### Accordion argomenti
```javascript
function toggleAcc(header){
  var item=header.parentElement;
  item.classList.toggle('open');
}
// HTML: <div class="acc-item"><div class="acc-header" onclick="toggleAcc(this)">
//   <span>icon</span><span class="acc-title">Titolo</span>
//   <span class="acc-chev">▼</span>
// </div><div class="acc-body"><div class="acc-inner">contenuto</div></div></div>
```

### Domande esame click-to-reveal
```javascript
function toggleDomanda(el){ el.classList.toggle('open'); }
// HTML: <div class="domanda-item hot" onclick="toggleDomanda(this)">
//   <div class="domanda-q">Domanda...</div>
//   <div class="domanda-ans">Risposta completa...</div>
// </div>
```

---

## 6. Template SH (sections HTML)

Le sezioni sono stringhe template in un oggetto `SH`:

```javascript
var SH={};
SH.home='...'; SH.percorsi='...'; SH.argomenti='...';
SH.glossario='...'; SH.flashcard='...'; SH.quiz='...';
SH.esami='...'; SH.simula='...'; SH.prof='...';
```

### SH.flashcard — struttura filtri
```html
<div class="fc-filter">
  <button class="filter-btn active" onclick="filterFC('Tutti',this)">Tutti</button>
  <button class="filter-btn" onclick="filterFC('Categoria1',this)">Categoria1</button>
  <!-- una per ogni cat unica nelle FLASHCARDS -->
</div>
<div class="flashcard-area">
  <div class="fc-meta">
    <span class="fc-counter" id="fcCounter"></span>
    <span class="fc-cat-badge" id="fcCat"></span>
  </div>
  <div class="flashcard" id="flashcard" onclick="flipCard()">
    <div class="fc-inner">
      <div class="fc-front">
        <div class="fc-label">DOMANDA</div>
        <div class="fc-question" id="fcQuestion"></div>
        <div class="fc-hint">Clicca per vedere la risposta</div>
      </div>
      <div class="fc-back">
        <div class="fc-label">RISPOSTA</div>
        <div class="fc-answer" id="fcAnswer"></div>
      </div>
    </div>
  </div>
  <div class="fc-actions">
    <button class="btn" onclick="rateCard('no')">✗ Rivedere</button>
    <button class="btn" onclick="prevCard()">← Prec.</button>
    <button class="btn" onclick="nextCard()">Succ. →</button>
    <button class="btn" onclick="shuffleFC()">🔀 Mescola</button>
  </div>
</div>
```

### SH.simula — struttura con timer
```html
<div class="section-header"><h2>Simulazione Esame</h2>
  <p>10 domande casuali. Alla fine vedi tempo e aree deboli.</p></div>
<div style="display:flex;align-items:center;justify-content:space-between;
  margin-bottom:12px;flex-wrap:wrap;gap:8px">
  <button class="btn primary" onclick="initSimula()">▶ Inizia Simulazione</button>
  <span id="simulaTimer" style="font-family:'IBM Plex Mono',monospace;
    font-size:14px;color:var(--accent)"></span>
</div>
<div class="quiz-wrap" id="simulaWrap"></div>
```

---

## 7. Sezione SH.esami — struttura consigliata

```
1. Alert box: formato esame (orale/scritto, durata, cosa valuta)
2. esame-card "Domande quasi certe" (chip chip-red):
   - domanda-item.hot × N (click-to-reveal con struttura risposta + parole chiave + errori comuni + follow-up)
3. esame-card "Domande frequenti" (chip chip-yellow):
   - domanda-item.mid × N
4. esame-card "Peso argomenti + strategia" (tabella .stbl + tip-box con 4 regole)
```

---

## 8. Sezione SH.prof — struttura consigliata

```
1. .prof-header (avatar emoji + nome + ruolo + chips tematici)
2. Grid 2 colonne:
   - .info-block "Contatti & Ricevimento"
   - .info-block "Posizione e Formazione"
3. .info-block "Materiali del Corso" (grid 2: manuale + letture)
4. .info-block "Programma Ufficiale" (lista moduli)
5. Grid 2 colonne:
   - .tip-box "Stile di insegnamento"
   - .tip-box "Cosa evitare all'orale" (lista ❌)
6. .info-block "Calendario e Appelli"
```

---

## 9. Pitfalls tecnici (errori noti)

### Windows — file grande → usare Python
Il file supera 200KB. Su Windows il bash heredoc fallisce con `ENAMETOOLONG`.
**Soluzione:** scrivere uno script `patch_NOME.py` e lanciarlo con:
```bash
python -X utf8 patch_NOME.py
```
Il flag `-X utf8` è obbligatorio per caratteri non-ASCII nel print.

### Validare JS dopo ogni patch
```bash
node -e "
var fs=require('fs');
var html=fs.readFileSync('file.html','utf8');
var js=html.slice(html.indexOf('<script>')+8, html.lastIndexOf('</script>'));
try{new Function(js);console.log('JS OK');}catch(ex){console.error(ex.message);}
"
```

### Apostrofi nei template string JS
I testi italiani contengono apostrofi. All'interno dei backtick template (`SH.xxx=\`...\``) usare `\'` nei tag HTML oppure evitare di usare la stessa virgoletta del JS circostante.
```javascript
// ✓ Corretto
SH.prof=`<p>L\'esame è orale</p>`;
// ✗ Sbagliato (rompe il template string)
SH.prof=`<p>L'esame è orale</p>`;
```

### Marker di ricerca per replace
Quando si fa `html.replace(OLD, NEW)` in Python, il marker deve essere univoco. Usare sempre 2-3 righe di contesto e non solo la prima riga.

### `domanda-item` — override CSS
Il file ha due definizioni di `.domanda-item`: una generica (linea ~39) e una aggiornata con `.open .domanda-ans{display:block}` e cursor. La seconda sovrascrive. Verificare che entrambe siano presenti.

### `navTo()` — tab senza data-id
I `.tab` non hanno `data-id`, usano `onclick="showSection(this,'ID')"`. La funzione `navTo(id)` scansiona `onclick` per trovare il tab giusto:
```javascript
function navTo(id){
  var btn=null;
  document.querySelectorAll('.tab').forEach(function(t){
    if(t.getAttribute('onclick')&&t.getAttribute('onclick').indexOf("'"+id+"'")>-1) btn=t;
  });
  showSection(btn,id);
  window.scrollTo(0,0);
}
```

---

## 10. Checklist finale prima di consegnare

- [ ] JS passa `node -e "new Function(js)"` senza errori
- [ ] `localStorage` usa prefisso univoco per l'esame (non `de_`)
- [ ] Ogni percorso ha campo `topics:[]` con id validi di TOPICS
- [ ] Ogni QUIZ entry ha campo `cat:` (stesso valore delle categorie Flashcard)
- [ ] Bottoni filtro Flashcard corrispondono esattamente alle `cat` in FLASHCARDS
- [ ] `SH.simula` ha pulsante "Inizia" + `id="simulaTimer"`
- [ ] Toast HTML presente prima di `</body>`
- [ ] Warn overlay presente con lista fonti parziali
- [ ] Nessuna dipendenza esterna (cdn, script src, img src esterne)
- [ ] File funziona aprendo direttamente `file://` nel browser

---

## 11. Sequenza di generazione ottimale (min token)

1. **Schema dati** — genera prima TOPICS, PERCORSI (senza steps), GLOSSARIO, categorie Flashcard
2. **Conferma struttura** — verifica con l'utente che argomenti e categorie siano corretti
3. **Contenuto pesante** — genera FLASHCARDS (il blocco più lungo), poi QUIZ
4. **Steps percorsi** — genera steps dettagliati per ogni percorso (8-9 step ciascuno)
5. **SH.argomenti** — accordion per ogni argomento con contenuto didattico ricco
6. **SH.esami + SH.prof** — le due sezioni descrittive
7. **Assembla** — incolla tutto nel template HTML base
8. **Patch aggiuntive** — connetti percorsi ↔ piano di studio, aggiungi `cat:` a QUIZ, valida JS

**Stima token per esame tipico:** ~80-100K token output totali.
Fare in sessioni separate per argomento se il contesto è vicino al limite.

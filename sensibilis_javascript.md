# Sensibilis — JavaScript-Logik (nur zum Lesen)

Stand: 24.07.2026 — Lead-Formular im Chat, Consent-Checkbox, Mikrofon entfernt, Supabase-Tabelle sensibilis_leads.

---

## Block 1 — Content Planer Demo (Zeile ~1453)

Simuliert den Demo-Ablauf im Content-Planer-Widget.

**Fixes in diesem Block:**
- Cancel-Token-Array `_cpdCancelTokens` löscht beim Neustart alle laufenden Timeouts
- Dadurch keine Race Conditions mehr wenn Demo während des Laufens neu gestartet wird

```javascript
(function(){
  var TI = '3 Nächte — 2 zahlen.';
  var TX = 'Nur im Juli: Drei Nächte buchen, zwei bezahlen...';
  var TG = '#Sommerspecial #Hotel #Juli #Auszeit #Angebot';
  var IMG = 'https://images.unsplash.com/photo-1542314831-068cd1dbfeeb?w=240&h=240&fit=crop&q=80';
  var INPUT = 'Sommerspecial Juli — entspannen & erholen';
  var running = false;
  var _cpdCancelTokens = []; // NEU: Cancel-Tokens für laufende Timeouts

  function getPos(id) { /* Position eines Elements relativ zum Wrapper */ }
  function moveCur(id, cb, waitAfter) { /* Cursor bewegen + Klick-Animation */ }
  function hiEl(id, color) { /* Element farblich hervorheben */ }
  function typeWrite(id, text, speed, cb) { /* Tipp-Animation */ }
  function resetDemo() { /* Alle Demo-Elemente zurücksetzen */ }

  window.cpdStart = function() {
    if (running) return;
    running = true;
    _cpdCancelTokens.forEach(clearTimeout); // NEU: Alte Timeouts abbrechen
    _cpdCancelTokens = [];
    resetDemo();
    // ... Cursor-Animation → Bereich wählen → Kanal wählen → Eingabe tippen → Generieren → Ergebnis
  };
})();
```

---

## Block 2 — KI Pass Schnellcheck (Zeile ~1951)

Dreistufiger Schnellcheck. Berechnet Risikoklasse 1–5 aus Datenkategorie, Einsatzart und Budget.

**Fix in diesem Block:**
- `sc2Los()` crashte wenn `D.find()` `undefined` zurückgab → sicherer Fallback auf Klasse 1

```javascript
(function(){
  var D = [
    { id:'a1', l:'Allgemeine Texte ohne Personenbezug',           k:1 },
    { id:'a2', l:'Mitarbeiter-interne Informationen',              k:2 },
    { id:'a3', l:'Kundendaten (Namen, Buchungen, Verträge)',       k:3 },
    { id:'a4', l:'Bewerber-, Personal- oder Finanzdaten',         k:4 },
    { id:'a5', l:'Gesundheits- oder andere sensible Daten',       k:5 },
  ];
  var E = [
    { id:'hr', l:'KI entscheidet/bewertet etwas Wichtiges für eine Person', h:'...' },
    { id:'cb', l:'KI kommuniziert direkt mit Kunden oder Bewerbern',         h:'...' },
    { id:'pr', l:'KI läuft nur intern, ohne Kontakt nach außen',             h:'...' },
    { id:'to', l:'Werkzeug für einzelne Mitarbeitende',                      h:'...' },
  ];
  var B = [
    { id:0, l:'Unter 50 €/Monat' },
    { id:1, l:'50–500 €/Monat' },
    { id:2, l:'500–5.000 €/Monat' },
    { id:3, l:'Über 5.000 €/Monat' },
  ];
  var KL = {
    1: { t:'Klasse 1 — Unkritisch',         d:'...', c:'#1a6b45' },
    2: { t:'Klasse 2 — Intern',             d:'...', c:'#2e7d32' },
    3: { t:'Klasse 3 — Vertraulich',        d:'...', c:'#B8842A' },
    4: { t:'Klasse 4 — Streng vertraulich', d:'...', c:'#d4700a' },
    5: { t:'Klasse 5 — Höchst sensibel',    d:'...', c:'#c0392b' },
  };
  var st = { d:[], e:null, b:null };

  // rd() / re() / rb() → bauen Checkbox- und Radio-Listen neu auf
  // step(n) → zeigt Schritt q1/q2/q3/res, aktualisiert Fortschrittsbalken
  // sc2Td / sc2Se / sc2Sb → globale Handler für onchange im generierten HTML

  window.sc2Los = function() {
    if (st.b === null) { err('Bitte ein Budget auswählen.'); return; }

    // NEU: sicherer Fallback — D.find() kann undefined zurückgeben
    var kl = 1;
    if (st.d.length) {
      var klassen = st.d.map(function(id) {
        var f = D.find(function(o) { return o.id === id; });
        return f ? f.k : 1; // Fallback auf 1 statt Absturz
      });
      kl = Math.max.apply(null, klassen);
    }
    var k = KL[kl] || KL[1]; // Fallback auch hier

    document.getElementById('sc2-rb').style.background = 'linear-gradient(135deg,' + k.c + ',' + k.c + 'bb)';
    document.getElementById('sc2-rn').textContent = kl;
    document.getElementById('sc2-rt').textContent = k.t;
    document.getElementById('sc2-rd').textContent = k.d;

    var isHR = st.e === 'hr';
    // Routing-Box: Hochrisiko → Prüftool 2 / Standard → Prüftool 1
    step('res');
  };

  window.sc2Reset = function() {
    st = { d:[], e:null, b:null };
    rd(); step(1);
    var kd = document.getElementById('kp-demo');
    if (kd) kd.style.display = 'none';
  };

  rd(); step(1);
})();
```

---

## Block 3 — Router, Canvas, Countdown, FAQ, Formular, Checkliste, KI Pass Demo, Chatbot (Zeile ~3479)

Haupt-Script-Block. Läuft nach dem gesamten HTML.

### Juli-Stempel

```javascript
(function(){
  var aktiveMonat = [7]; // Monatszahl: 7 = Juli
  if (aktiveMonat.indexOf(new Date().getMonth() + 1) !== -1) {
    document.getElementById('juli-stempel').style.display = 'flex';
  }
})();
```

---

### Router + SEO-Meta

**Fixes:**
- `history.pushState` → Browser-Zurück-Taste funktioniert jetzt
- `popstate`-Listener → Zurück-Navigation springt zur richtigen Seite
- Mobile-Menü-Links schließen das Menü automatisch

```javascript
var SEO_META = {
  'blog-1':     { t:'EU AI Act für kleine Unternehmen 2026 | Sensibilis', d:'...' },
  'blog-2':     { t:'5 KI-Tools die wirklich Zeit sparen (2026) | Sensibilis', d:'...' },
  'blog-3':     { t:'ChatGPT & DSGVO 2026 | Sensibilis', d:'...' },
  'blog-4':     { t:'Sensibilis — KI-Beratung | Bad Bodenteich', d:'...' },
  'blog':       { t:'Blog & Insights | Sensibilis', d:'...' },
  'kipass':     { t:'KI Pass — DSGVO & EU AI Act Dokumentation | Sensibilis', d:'...' },
  'home':       { t:'Sensibilis — KI-Services für Unternehmen | Annett Mende', d:'...' },
  'checkliste': { t:'EU AI Act Selbstcheck für Ihren Betrieb | Sensibilis', d:'...' },
};

function nav(id) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  var pg = document.getElementById('page-' + id);
  if (pg) pg.classList.add('active');
  if (id === 'checkliste') chkReset();
  if (id === 'kipass') {
    var kd = document.getElementById('kp-demo');
    if (kd) kd.style.display = 'none';
  }
  document.querySelectorAll('.nav-links a[data-page]').forEach(function(a) {
    a.classList.toggle('active', a.dataset.page === id);
  });
  if (window.location.hash !== '#' + id) history.pushState(null, null, '#' + id); // NEU
  window.scrollTo({ top: 0, behavior: 'instant' });
  var m = SEO_META[id];
  if (m) {
    document.title = m.t;
    var dm = document.querySelector('meta[name="description"]');
    if (dm) dm.setAttribute('content', m.d);
  }
}

// NEU: Zurück-Taste im Browser
window.addEventListener('popstate', function() {
  var h = window.location.hash.replace('#', '');
  if (h) nav(h);
});

// Ham-Button
document.getElementById('ham-btn').addEventListener('click', function() {
  const open = document.getElementById('mob-menu').classList.toggle('open');
  this.setAttribute('aria-expanded', open);
  this.setAttribute('aria-label', open ? 'Menü schließen' : 'Menü öffnen');
});

// closeMob: NEU mit aria-Sync
function closeMob() {
  var menu = document.getElementById('mob-menu');
  var btn  = document.getElementById('ham-btn');
  if (menu.classList.contains('open')) {
    menu.classList.remove('open');
    if (btn) {
      btn.setAttribute('aria-expanded', 'false');
      btn.setAttribute('aria-label', 'Menü öffnen');
    }
  }
}

window.addEventListener('DOMContentLoaded', function() {
  var h = window.location.hash.replace('#', '');
  if (h) nav(h);
  // NEU: Mobile Menü-Links schließen Menü automatisch
  document.querySelectorAll('#mob-menu a').forEach(function(link) {
    link.addEventListener('click', function() { closeMob(); });
  });
});
```

---

### Canvas-Partikel

**Fix:** Resize-Event debounced auf 120ms — verhindert Performance-Spikes bei schnellem Resize.

```javascript
(function(){
  const c = document.getElementById('hero-canvas');
  if (!c) return;
  if (window.innerWidth < 768 || window.matchMedia('(prefers-reduced-motion:reduce)').matches) {
    c.style.display = 'none'; return;
  }
  const ctx = c.getContext('2d');
  const hero = c.closest('section') || c.parentElement;
  let W, H, pts = [];

  function resize() { W = c.width = hero.offsetWidth; H = c.height = hero.offsetHeight; }
  resize();
  var _resizeT;
  window.addEventListener('resize', function() { // NEU: debounced
    clearTimeout(_resizeT); _resizeT = setTimeout(resize, 120);
  });

  // 90 Punkte mit zufälliger Position/Geschwindigkeit, prallen an Kanten ab
  // Verbindungslinien bei Abstand < 110px, Deckkraft nimmt mit Abstand ab
  // requestAnimationFrame-Loop
})();
```

---

### Countdown

```javascript
function tick() {
  const end  = new Date('2026-07-31T23:59:59');
  const diff = end - new Date();
  if (diff < 0) return;
  const d = Math.floor(diff / 86400000);
  const h = Math.floor((diff % 86400000) / 3600000);
  const m = Math.floor((diff % 3600000)  / 60000);
  ['cd-d','cd-h','cd-m'].forEach((id, i) => {
    const el = document.getElementById(id);
    if (el) el.textContent = String([d, h, m][i]).padStart(2, '0');
  });
}
tick(); setInterval(tick, 60000);
```

---

### FAQ, Formular, Checkliste

```javascript
// FAQ — Accordion: immer nur ein Item offen
function toggleFaq(btn) {
  const item = btn.closest('.faq-item');
  const wasOpen = item.classList.contains('open');
  document.querySelectorAll('.faq-item.open').forEach(i => i.classList.remove('open'));
  if (!wasOpen) item.classList.add('open');
}

// Formular — Hinweis: kein echter Versand, wird beim Go-Live ergänzt (Formspree)
function handleForm(e) {
  e.preventDefault();
  document.getElementById('contact-success').style.display = 'block';
  e.target.reset();
}

// Checkliste — Ergebnis nach 9 Fragen
function chkShowResult() {
  var score = document.querySelectorAll('#page-checkliste .chk-item.checked').length;
  // ≥7 → gut aufgestellt (Navy) / ≥4 → Lücken (Gold) / <4 → Handlungsbedarf (Rot)
}
function chkReset() {
  document.querySelectorAll('#page-checkliste .chk-item').forEach(el => el.classList.remove('checked'));
  document.getElementById('chk-result').style.opacity = '0';
  document.getElementById('chk-result').style.display = 'none';
  document.getElementById('chk-cta').style.display = 'none';
}
```

---

### KI Pass Demo

**Fixes:**
- `kpTimer` wird jetzt in `kpType` gesetzt und in `kpDemoStart` vor Neustart gelöscht
- Checks in Scene 1 per `createElement` + `textContent` statt `innerHTML`

```javascript
var kpTimer = null;

function kpScene(i) {
  // Szene 0/1/2 umschalten + Tab-Styling
}

function kpType(elId, text, speed, cb) {
  var el = document.getElementById(elId);
  if (!el) return;
  el.textContent = '';
  var i = 0;
  kpTimer = setInterval(function() { // NEU: in kpTimer speichern
    el.textContent += text[i]; i++;
    if (i >= text.length) { clearInterval(kpTimer); kpTimer = null; if (cb) cb(); }
  }, speed || 60);
}

function kpDemoStart() {
  if (kpTimer) { clearInterval(kpTimer); kpTimer = null; } // NEU: alten Timer stoppen

  var kl = document.getElementById('sc2-rn') ? parseInt(document.getElementById('sc2-rn').textContent) || 3 : 3;
  var KL = { 1:{c:'#1a6b45',...}, 2:{...}, 3:{c:'#B8842A',...}, 4:{...}, 5:{c:'#c0392b',...} };
  var k = KL[kl] || KL[3];

  // Ablauf:
  // Scene 0: kpType → Tool-Name → Tool-Zweck eintippen
  // Scene 1: 4 Compliance-Checks erscheinen mit je 600ms Verzögerung
  //          → createElement + textContent statt innerHTML (NEU)
  // Scene 2: conic-gradient Ring animiert 0→pct in 20ms-Schritten
  //          → gestaffelte Fade-ins: subs(+400ms), good(+700ms), open(+950ms), plan(+1200ms)

  var checks = [
    { t:'Auftragsverarbeitungsvertrag (AVV)',     ok:true  },
    { t:'DSGVO-konforme Einstellungen aktiviert', ok:true  },
    { t:'Daten-Training deaktiviert',             ok:false },
    { t:'EU AI Act Risikoklasse bestimmt',        ok:true  },
  ];
  checks.forEach(function(ch, idx) {
    setTimeout(function() {
      var d = document.createElement('div');
      d.style.cssText = 'display:flex;align-items:center;gap:10px;...opacity:0;transition:opacity .4s;';
      // NEU: createElement statt innerHTML
      var ic = document.createElement('span');
      ic.style.color = ch.ok ? '#4caf50' : '#ef5350';
      ic.textContent = ch.ok ? '✓' : '✗';
      var lb = document.createElement('span');
      lb.textContent = ch.t;
      d.appendChild(ic); d.appendChild(lb);
      document.getElementById('kp-checks').appendChild(d);
      setTimeout(function() { d.style.opacity = '1'; }, 50);
    }, idx * 600);
  });
}
```

---

### Chatbot

**Fix:** Enter-Taste sendet die Nachricht ab.

```javascript
// NEU: Enter-Taste im Chatfeld
document.addEventListener('DOMContentLoaded', function() {
  var ci = document.getElementById('chat-in');
  if (ci) ci.addEventListener('keydown', function(e) {
    if (e.key === 'Enter') { e.preventDefault(); sendChat(); }
  });
});

// Wissensbasis: 21 Regex-Antwort-Paare
const kb = [
  [/was ist.*ki pass|ki pass.*was|erkl.r.*ki pass/i,        'Der KI Pass ist ein digitales Dokumentations-Tool...'],
  [/schnellcheck|selbstcheck|test.*ki|ki.*test|checkliste/i, 'Der Schnellcheck ist kostenlos — drei Fragen, zwei Minuten...'],
  [/preis|kosten|was kostet|wie teuer|39|juli.*aktion/i,    'Im Juli gibt es den KI Pass für 39 €...'],
  [/eu ai act|ai act|ki.gesetz|ki.regulierung/i,            'Der EU AI Act ist seit 2024 in Kraft...'],
  [/frist|deadline|wann|bis wann|pflicht/i,                 'Seit Februar 2025 gelten die Verbote für unannehmbares Risiko...'],
  [/dsgvo|datenschutz|personenbezogen|avv/i,                'Wer ChatGPT für Kundentexte nutzt, braucht einen AVV...'],
  [/chatgpt|gpt|openai/i,                                   'ChatGPT ist das meistgenutzte KI-Tool in kleinen Betrieben...'],
  [/copilot|microsoft|office.*ki/i,                         'Microsoft Copilot hat einen AVV bereits standardmäßig eingebettet...'],
  [/risikoklasse|klasse|einstufung|hochrisiko/i,            'Der EU AI Act unterscheidet vier Stufen...'],
  [/content.planer|redaktionsplan|social media.*ki/i,       'Der Content Planer ist bereits verfügbar — 490,- € einmalig inkl. 12 Monate Hosting...'],
  [/website.check|webcheck|website.*prüf/i,                 'Der Website-Check ist in Entwicklung...'],
  [/handwerk|werkstatt|tischler|schreiner/i,                'Im Handwerk steckt mehr KI-Potenzial als viele denken...'],
  [/hotel|gastronomie|restaurant|rezeption/i,               'Hotellerie und Gastronomie kenne ich aus eigener Erfahrung...'],
  [/wer bist|wer sind|annett|sensibilis.*wer/i,             'Ich bin Annett Mende...'],
  [/wo|region|niedersachsen|uelzen|gifhorn|bad bodenteich/i,'Sensibilis ist in Bad Bodenteich ansässig...'],
  [/wie.*anfang|wo.*anfang|erst.*schritt|einstieg/i,        'Am besten fangen Sie mit dem Schnellcheck an...'],
  [/aufwand|dauer|wie lang|wie viel zeit/i,                 'Der Schnellcheck dauert zwei Minuten...'],
  [/beratung|gespräch|termin|kontakt|mail/i,                'Das Erstgespräch ist ohne Berechnung...'],
  [/sicher|vertrauen|daten.*weg|training.*daten/i,          'Die meisten KI-Anbieter trainieren ihre Modelle mit Eingaben...'],
  [/zukunft|geplant|demnächst|was kommt/i,                  'Verfügbar: KI Pass, Schnellcheck, Content Planer. In Entwicklung: Website-Check, DMS...'],
  [/ki|künstliche intelligenz|ai\b/i,                       'KI ist längst kein Zukunftsthema mehr...'],
  [/wetter|fußball|bundesliga|rezept|aktien|börse|bitcoin/i,'Das liegt außerhalb meines Fachgebiets — ich bin auf KI-Beratung spezialisiert...'],
];

// Chip-Klick: Chips ausblenden, Text ins Eingabefeld, sendChat() aufrufen
function chipClick(text) {
  var chips = document.getElementById('chat-chips');
  if (chips) chips.style.display = 'none';
  document.getElementById('chat-in').value = text;
  sendChat();
}

function sendChat() {
  const inp  = document.getElementById('chat-in');
  const q    = inp.value.trim(); if (!q) return;
  var chips  = document.getElementById('chat-chips');
  if (chips) chips.style.display = 'none';       // Chips beim ersten Absenden ausblenden
  const msgs = document.getElementById('chat-msgs');
  msgs.innerHTML += `<div class="msg msg-user">${q.replace(/</g,'&lt;')}</div>`;
  inp.value = '';
  const matchIdx = kb.findIndex(([rx]) => rx.test(q));
  const match    = matchIdx >= 0 ? kb[matchIdx] : null;
  const reply    = match ? match[1] : 'Das beantworte ich am besten persönlich — schreiben Sie Annett an amen1@t-online.de.';
  const topic    = matchIdx >= 0 ? kbTopics[matchIdx] || null : null;
  const isLead   = /beratung|gespräch|termin|kontakt|schreib|mail|telefon|preis|kosten|wie teuer/i.test(q);
  // Tipp-Indikator (3 springende Punkte) für 700 ms
  const typId = 'typ' + Date.now();
  msgs.innerHTML += `<div class="chat-typing" id="${typId}"><span></span><span></span><span></span></div>`;
  msgs.scrollTop = msgs.scrollHeight;
  setTimeout(() => {
    const typ = document.getElementById(typId);
    if (typ) typ.remove();
    const cta = isLead ? '<br><button class="chat-cta" onclick="nav(\'kontakt\')">Direkt anfragen →</button>' : '';
    msgs.innerHTML += `<div class="msg msg-bot">${reply}${cta}</div>`;
    msgs.scrollTop = msgs.scrollHeight;
    // Chat-Tracking direkt an Supabase (Render-Server umgangen)
    try {
      var sid = sessionStorage.getItem('snb_sid') || 'anon';
      fetch('https://qrpaeeglfpfywunvvgkq.supabase.co/rest/v1/sensibilis_chats', { ... });
    } catch(e) {}
  }, 700);
}
```

---

## Nav-Active Styling (15.07.2026)

Aktiver Menüpunkt war optisch kaum erkennbar (`color:var(--iv)` + minimaler Hintergrund).

**Lösung:** Hover und Active getrennt, Active deutlich hervorgehoben:

```css
.nav-links a:hover { color:var(--iv); background:rgba(255,255,255,.07); }
.nav-links a.active { color:var(--g); background:rgba(184,146,74,.12); border-bottom:2px solid var(--g); font-weight:600; }
```

---

## Countdown entfernt (15.07.2026)

`tick()`-Funktion und `#countdown`-CSS komplett entfernt — war für die Juli-Aktion vorgesehen, HTML-Element existierte nicht mehr. Kein aktiver Code mehr.

---

## BlogPosting JSON-LD verschoben (15.07.2026)

Die drei `BlogPosting`-Einträge aus dem JSON-LD der `sensibilis.html` entfernt. Sie lagen doppelt vor — einmal auf der Startseite, einmal auf den Unterseiten. Startseite enthält jetzt nur noch `LocalBusiness` + `FAQPage`. Die Blog-Unterseiten (`blog-1.html`, `blog-2.html`, `blog-3.html`) haben ihr eigenes korrektes `BlogPosting`-JSON-LD.

---

## Bugfix: Juli-Stempel Null-Sicherung (15.07.2026)

`document.getElementById('juli-stempel')` wird jetzt in Variable `stempel` gespeichert und vor dem Zugriff auf `.style` geprüft (`if(stempel&&...)`). Verhindert TypeError falls Element entfernt wird.

---

## Bugfix: Handbücher Zeichenfehler (15.07.2026)

`Handbu&#807;cher` (Combining Cedilla, falsch) ersetzt durch `Handb&uuml;cher` (korrektes ü).

---

## Offene Punkte

- **Kontaktformular**: Aktuell kein echter Versand. Beim Go-Live Formspree einbinden.

---

## Bild-Auslagerung (15.07.2026)

Das Foto von Annett Mende war als Base64-String direkt im HTML eingebettet (~372 KB inline).

**Problem:** Kein Browser-Caching möglich, HTML-Datei unnötig aufgebläht.

**Lösung:** Base64 aus `sensibilis.html` und `index.html` entfernt, Bild als separate Datei abgelegt:

```
Sensibilis-Ki/
  img/
    annett-mende.png   ← freigestelltes Foto (Bild_A._Mende-removebg-preview.png)
  sensibilis.html      ← src="img/annett-mende.png" (3 Stellen)
  index.html           ← src="img/annett-mende.png" (3 Stellen)
```

Gepusht als Commit `eeb87ae` auf `AnMe15/Sensibilis-Ki`.

---

## Mikrofon-Funktion entfernt (24.07.2026)

`startMic()`, `var _mic`, CSS für `#chat-mic` und `@keyframes micpulse` vollständig entfernt. Mikrofon-Button aus dem Chat-Eingabefeld entfernt. Begründung: Spracherkennungsqualität nicht überzeugend genug für den Produktivbetrieb.

---

## Lead-Formular im Chat (24.07.2026)

Bei Beratungs-Themen (`Beratung & Kontakt` / `Beratung Ablauf`) erscheint nach der Bot-Antwort ein Inline-Formular direkt im Chat-Bubble.

**Supabase-Tabelle:** `sensibilis_leads` (neu angelegt) mit Spalten: `id`, `created_at`, `name`, `email`, `session_id`, `source`, `newsletter_consent`.

**CSS:** `.chat-lead-form`, `.lead-input`, `.lead-submit`

**Funktion `submitLead(fid)`:**

```javascript
var _FORMSPREE = 'YOUR_FORM_ID'; // deaktiviert — DSGVO-Bedenken (US-Server)
function submitLead(fid) {
  var name    = document.getElementById('lf-name-'+fid).value.trim();
  var email   = document.getElementById('lf-email-'+fid).value.trim();
  if (!email) { /* E-Mail-Feld rot markieren + fokussieren */ return; }
  var consent = document.getElementById('lf-consent-'+fid)
                  ? document.getElementById('lf-consent-'+fid).checked : false;
  // Formular durch Bestätigungstext ersetzen
  document.getElementById('lead-form-'+fid).innerHTML = '<p>Danke [Name]! Annett meldet sich bei Ihnen.</p>';
  // Speichern in Supabase sensibilis_leads (inkl. newsletter_consent)
  fetch('https://qrpaeeglfpfywunvvgkq.supabase.co/rest/v1/sensibilis_leads', { method:'POST', ... });
  // Formspree deaktiviert bis DSGVO-konformer Versandweg steht (ab 1.8. Render)
}
```

**Consent-Checkbox:** Optional, unter "Anfrage senden". Text: *"Ja, ich möchte über neue KI-Tools und Angebote informiert werden."* Wert wird als `newsletter_consent` (Boolean) in Supabase gespeichert.

**Nächster Schritt (ab 1.8.2026):** E-Mail-Benachrichtigung bei neuem Lead über Render-Backend (eigener SMTP, kein Drittanbieter).

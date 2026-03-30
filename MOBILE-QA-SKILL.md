---
name: mobile-qa
description: Mobile QA Testing mit echten Geräten via Mobilerun API. Nutze wenn du eine Website oder App auf einem echten Android/iOS Gerät testen willst. Iterativer Cycle - Screenshot → Analyse → Fix → Verify. Für responsive Design, Mobile UX und visuelles Testing.
---

# Mobile QA mit Mobilerun

Automatisiertes Mobile QA Testing auf echten Geräten. Kein Emulator, keine DevTools — echtes Handy, echte Screenshots.

## Setup

### Credentials
```bash
# API Key und Device ID müssen vorhanden sein
cat ~/.config/droidrun/config.json
# {"api_key": "dr_sk_...", "device_id": "..."}
```

### Device prüfen
```bash
API_KEY=$(python3 -c "import json;print(json.load(open('$HOME/.config/droidrun/config.json'))['api_key'])")
curl -s "https://api.mobilerun.ai/v1/devices" -H "Authorization: Bearer $API_KEY"
# State muss "ready" sein
```

## API Referenz

### Task erstellen
```bash
# WICHTIG: Kein llmModel angeben — API wählt automatisch
curl -s -X POST "https://api.mobilerun.ai/v1/tasks" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "deviceId": "DEVICE_ID",
    "task": "Open Chrome and go to https://example.com then take a screenshot"
  }'
# Response: {"id": "TASK_ID", "streamUrl": "wss://..."}
```

### Task Status pollen
```bash
curl -s "https://api.mobilerun.ai/v1/tasks/$TASK_ID" \
  -H "Authorization: Bearer $API_KEY"
# task.succeeded: true/false/null (null = noch am laufen)
```

### Screenshots holen (Zwei Schritte!)
```bash
# Schritt 1: Signed URL holen
SIGNED_URL=$(curl -s "https://api.mobilerun.ai/v1/tasks/$TASK_ID/screenshots/$INDEX" \
  -H "Authorization: Bearer $API_KEY" | python3 -c "import json,sys;print(json.load(sys.stdin)['url'])")

# Schritt 2: Bild downloaden
curl -sL "$SIGNED_URL" -o screenshot.png
```

### Alle Screenshots eines Tasks
```bash
# Screenshot-Indices aus Trajectory extrahieren
curl -s "https://api.mobilerun.ai/v1/tasks/$TASK_ID" -H "Authorization: Bearer $API_KEY" | \
  python3 -c "import json,sys;[print(ev['data']['index']) for ev in json.load(sys.stdin)['task']['trajectory'] if ev['event']=='ScreenshotEvent']"
```

## QA Workflow

### Der Cycle: Screenshot → Analyse → Fix → Verify

```
┌─────────────┐
│ 1. Screenshot│──→ Task an Gerät: "Öffne URL, scrolle durch"
└──────┬──────┘
       ▼
┌─────────────┐
│ 2. Download  │──→ Screenshots von allen Sections holen
└──────┬──────┘
       ▼
┌─────────────┐
│ 3. Analyse   │──→ Screenshots analysieren: Layout, Overflow, Touch Targets
└──────┬──────┘
       ▼
┌─────────────┐
│ 4. Fix       │──→ CSS/HTML anpassen
└──────┬──────┘
       ▼
┌─────────────┐
│ 5. Reload    │──→ nginx/Server reloaden
└──────┬──────┘
       ▼
┌─────────────┐
│ 6. Verify    │──→ Neuer Screenshot, prüfen ob Fix wirkt
└──────┬──────┘
       ▼
    Repeat bis alles passt (min. 3 Cycles)
```

### Cycle 1: Voller Page Scroll
```bash
# Seite öffnen und komplett durchscrollen
TASK="Open Chrome and navigate to $URL. Wait for full load. Then scroll down slowly through the entire page from top to bottom."

# → Screenshots analysieren für: Überlappungen, Overflow, kaputte Grids, unleserlichen Text
```

### Cycle 2: Gezielte Sections
```bash
# Spezifische Problem-Sections checken
TASK="Scroll to the section with the comparison table. Take a screenshot. Then scroll to the footer and take a screenshot."
```

### Cycle 3: Interaktions-Test
```bash
# Buttons, Navigation, Dropdowns testen
TASK="Tap the hamburger menu icon in the top right. Take a screenshot of the open menu. Then close it and scroll to a section with buttons."
```

## Mobile QA Checkliste

### Layout
- [ ] Kein horizontaler Overflow (kein seitliches Scrollen)
- [ ] Grids stacken auf Mobile (1-spaltig statt 3-spaltig)
- [ ] Cards haben volle Breite
- [ ] Tabellen haben horizontal-scroll oder passen sich an
- [ ] Bilder/Videos behalten Aspect Ratio
- [ ] Kein Content wird abgeschnitten

### Navigation
- [ ] Hamburger Menu funktioniert
- [ ] Sticky/Fixed Bars überlappen NICHT den Content
- [ ] Alle Links sind tappbar (min. 44x44px Touch Target)
- [ ] Navigation schließt nach Tap

### Text
- [ ] Alle Headlines lesbar (min. 20px auf Mobile)
- [ ] Body Text min. 14px
- [ ] Ausreichend Kontrast
- [ ] Kein Text wird abgeschnitten oder überlappt
- [ ] Code-Blöcke brechen nicht aus dem Viewport

### Interaktion
- [ ] Buttons haben genug Padding für Finger-Tap
- [ ] Formulare sind benutzbar
- [ ] Dropdowns/Accordions funktionieren
- [ ] Scroll-Animationen flüssig

### Typische Mobile-Bugs
- [ ] `position:fixed` Elemente überlappen nicht
- [ ] `overflow:hidden` auf Carousel-Containern
- [ ] `max-width:100vw` auf äußeren Containern
- [ ] `grid-template-columns:1fr` im Media Query
- [ ] ASCII Art / Monospace-Blöcke hidden auf Mobile
- [ ] Testimonial-Karussells begrenzt auf Viewport-Breite

## Beispiel: Kompletter QA Run

```bash
#!/bin/bash
API_KEY="$(python3 -c "import json;print(json.load(open('$HOME/.config/droidrun/config.json'))['api_key'])")"
DEVICE_ID="$(python3 -c "import json;print(json.load(open('$HOME/.config/droidrun/config.json'))['device_id'])")"
URL="https://example.com"
OUTPUT_DIR="./mobile-qa-$(date +%Y%m%d-%H%M)"
mkdir -p "$OUTPUT_DIR"

# Task erstellen
TASK_ID=$(curl -s -X POST "https://api.mobilerun.ai/v1/tasks" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"deviceId\":\"$DEVICE_ID\",\"task\":\"Open Chrome and go to $URL. Scroll through the entire page slowly.\"}" \
  | python3 -c "import json,sys;print(json.load(sys.stdin)['id'])")

echo "Task: $TASK_ID"

# Pollen bis fertig
while true; do
  sleep 10
  SUCCEEDED=$(curl -s "https://api.mobilerun.ai/v1/tasks/$TASK_ID" \
    -H "Authorization: Bearer $API_KEY" \
    | python3 -c "import json,sys;print(json.load(sys.stdin).get('task',{}).get('succeeded','pending'))")
  echo "Status: $SUCCEEDED"
  [ "$SUCCEEDED" = "True" ] || [ "$SUCCEEDED" = "False" ] && break
done

# Screenshots holen
INDICES=$(curl -s "https://api.mobilerun.ai/v1/tasks/$TASK_ID" \
  -H "Authorization: Bearer $API_KEY" \
  | python3 -c "import json,sys;[print(ev['data']['index']) for ev in json.load(sys.stdin)['task']['trajectory'] if ev['event']=='ScreenshotEvent']")

for i in $INDICES; do
  SIGNED=$(curl -s "https://api.mobilerun.ai/v1/tasks/$TASK_ID/screenshots/$i" \
    -H "Authorization: Bearer $API_KEY" \
    | python3 -c "import json,sys;print(json.load(sys.stdin)['url'])")
  curl -sL "$SIGNED" -o "$OUTPUT_DIR/screenshot-$i.png"
  echo "Downloaded: $OUTPUT_DIR/screenshot-$i.png"
done

echo "Done. Screenshots in $OUTPUT_DIR/"
```

## Bekannte Gotchas

- `llmModel` NICHT angeben — API wählt automatisch ein verfügbares Model
- Screenshots sind **zwei Schritte**: erst signed URL, dann Download
- `api.mobilerun.ai` nutzen, nicht `api.droidrun.ai`
- Device muss `state: ready` sein
- Tasks dauern 20-90 Sekunden je nach Komplexität
- Chrome auf dem Gerät zeigt manchmal Translate-Bar — ignorieren
- Gerät behält Browser-State zwischen Tasks — "Reload" statt "Open" nutzen wenn Seite schon offen

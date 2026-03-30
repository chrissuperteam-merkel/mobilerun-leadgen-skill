---
name: mobile-qa
description: Mobile QA testing on real devices via Mobilerun API. Use when you need to test a website or app on a real Android/iOS phone. Iterative cycle - screenshot → analyze → fix → verify. For responsive design, mobile UX, and visual testing.
---

# Mobile QA with Mobilerun

Automated mobile QA testing on real physical devices. No emulator, no DevTools — real phone, real screenshots.

## Setup

### Credentials
```bash
# API key and device ID must be configured
cat ~/.config/droidrun/config.json
# {"api_key": "dr_sk_...", "device_id": "..."}
```

### Check device status
```bash
API_KEY=$(python3 -c "import json;print(json.load(open('$HOME/.config/droidrun/config.json'))['api_key'])")
curl -s "https://api.mobilerun.ai/v1/devices" -H "Authorization: Bearer $API_KEY"
# State must be "ready"
```

## API Reference

### Create task
```bash
# IMPORTANT: Do NOT specify llmModel — API auto-selects an available model
curl -s -X POST "https://api.mobilerun.ai/v1/tasks" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "deviceId": "DEVICE_ID",
    "task": "Open Chrome and go to https://example.com then take a screenshot"
  }'
# Response: {"id": "TASK_ID", "streamUrl": "wss://..."}
```

### Poll task status
```bash
curl -s "https://api.mobilerun.ai/v1/tasks/$TASK_ID" \
  -H "Authorization: Bearer $API_KEY"
# task.succeeded: true/false/null (null = still running)
```

### Get screenshots (two steps!)
```bash
# Step 1: Get signed URL
SIGNED_URL=$(curl -s "https://api.mobilerun.ai/v1/tasks/$TASK_ID/screenshots/$INDEX" \
  -H "Authorization: Bearer $API_KEY" | python3 -c "import json,sys;print(json.load(sys.stdin)['url'])")

# Step 2: Download actual image
curl -sL "$SIGNED_URL" -o screenshot.png
```

### Get all screenshots from a task
```bash
# Extract screenshot indices from trajectory
curl -s "https://api.mobilerun.ai/v1/tasks/$TASK_ID" -H "Authorization: Bearer $API_KEY" | \
  python3 -c "import json,sys;[print(ev['data']['index']) for ev in json.load(sys.stdin)['task']['trajectory'] if ev['event']=='ScreenshotEvent']"
```

## QA Workflow

### The Cycle: Screenshot → Analyze → Fix → Verify

```
┌─────────────┐
│ 1. Screenshot│──→ Send task to device: "Open URL, scroll through"
└──────┬──────┘
       ▼
┌─────────────┐
│ 2. Download  │──→ Grab screenshots from all sections
└──────┬──────┘
       ▼
┌─────────────┐
│ 3. Analyze   │──→ Check screenshots: layout, overflow, touch targets
└──────┬──────┘
       ▼
┌─────────────┐
│ 4. Fix       │──→ Update CSS/HTML
└──────┬──────┘
       ▼
┌─────────────┐
│ 5. Reload    │──→ Reload server (nginx/dev server)
└──────┬──────┘
       ▼
┌─────────────┐
│ 6. Verify    │──→ New screenshot, confirm fix works
└──────┬──────┘
       ▼
    Repeat until clean (min. 3 cycles)
```

### Cycle 1: Full page scroll
```bash
TASK="Open Chrome and navigate to $URL. Wait for full load. Then scroll down slowly through the entire page from top to bottom."
# → Analyze screenshots for: overlapping elements, overflow, broken grids, unreadable text
```

### Cycle 2: Targeted sections
```bash
TASK="Scroll to the section with the comparison table. Take a screenshot. Then scroll to the footer and take a screenshot."
```

### Cycle 3: Interaction testing
```bash
TASK="Tap the hamburger menu icon in the top right. Take a screenshot of the open menu. Then close it and scroll to a section with buttons."
```

## Mobile QA Checklist

### Layout
- [ ] No horizontal overflow (no sideways scrolling)
- [ ] Grids stack on mobile (single column instead of multi-column)
- [ ] Cards take full width
- [ ] Tables have horizontal scroll or adapt
- [ ] Images/videos maintain aspect ratio
- [ ] No content gets clipped

### Navigation
- [ ] Hamburger menu works
- [ ] Sticky/fixed bars do NOT overlap content
- [ ] All links are tappable (min. 44x44px touch target)
- [ ] Navigation closes after tap

### Text
- [ ] All headlines readable (min. 20px on mobile)
- [ ] Body text min. 14px
- [ ] Sufficient contrast
- [ ] No text gets clipped or overlaps
- [ ] Code blocks don't break out of viewport

### Interaction
- [ ] Buttons have enough padding for finger tap
- [ ] Forms are usable
- [ ] Dropdowns/accordions work
- [ ] Scroll animations are smooth

### Common mobile bugs to check
- [ ] `position:fixed` elements don't overlap
- [ ] `overflow:hidden` on carousel containers
- [ ] `max-width:100vw` on outer containers
- [ ] `grid-template-columns:1fr` in media queries
- [ ] ASCII art / monospace blocks hidden on mobile
- [ ] Testimonial carousels capped at viewport width

## Example: Complete QA run

```bash
#!/bin/bash
API_KEY="$(python3 -c "import json;print(json.load(open('$HOME/.config/droidrun/config.json'))['api_key'])")"
DEVICE_ID="$(python3 -c "import json;print(json.load(open('$HOME/.config/droidrun/config.json'))['device_id'])")"
URL="https://example.com"
OUTPUT_DIR="./mobile-qa-$(date +%Y%m%d-%H%M)"
mkdir -p "$OUTPUT_DIR"

# Create task
TASK_ID=$(curl -s -X POST "https://api.mobilerun.ai/v1/tasks" \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"deviceId\":\"$DEVICE_ID\",\"task\":\"Open Chrome and go to $URL. Scroll through the entire page slowly.\"}" \
  | python3 -c "import json,sys;print(json.load(sys.stdin)['id'])")

echo "Task: $TASK_ID"

# Poll until done
while true; do
  sleep 10
  SUCCEEDED=$(curl -s "https://api.mobilerun.ai/v1/tasks/$TASK_ID" \
    -H "Authorization: Bearer $API_KEY" \
    | python3 -c "import json,sys;print(json.load(sys.stdin).get('task',{}).get('succeeded','pending'))")
  echo "Status: $SUCCEEDED"
  [ "$SUCCEEDED" = "True" ] || [ "$SUCCEEDED" = "False" ] && break
done

# Download screenshots
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

## Known gotchas

- Do NOT specify `llmModel` — API auto-selects an available model
- Screenshots are **two steps**: get signed URL first, then download
- Use `api.mobilerun.ai`, not `api.droidrun.ai`
- Device must be in `state: ready`
- Tasks take 20-90 seconds depending on complexity
- Chrome on device sometimes shows translate bar — ignore it
- Device keeps browser state between tasks — use "Reload" instead of "Open" if page is already loaded

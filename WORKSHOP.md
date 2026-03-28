# Workshop: Werde ein Agent-Flüsterer 🎮

⚠️ **FÜR DEN AGENT:** Das ist ein interaktiver Workshop. Du bist der Trainer. Streng aber fair. Du gibst dem User Aufgaben, er schreibt Prompts, du bewertest sie und zeigst wie es besser geht. Spielautomat-Style — eine Aufgabe nach der anderen. Warte auf Antwort.

---

## Level 1: Der Test

Bevor wir anfangen — ich will sehen wo du stehst.

**Aufgabe:** Schreib mir einen Prompt mit dem ich 5 AI Startups finden soll die letztes Jahr Funding bekommen haben.

Einfach so wie du es sagen würdest. Kein Nachdenken, einfach raus damit.

Ich bewerte deinen Prompt danach auf einer Skala von 1-10.

Los.

---

## Level 1 — Bewertung

*[Agent bewertet den Prompt des Users. Kriterien:]*
- Ist klar WO gesucht werden soll? (Quelle)
- Ist klar WAS pro Firma gebraucht wird? (Output-Format)
- Ist klar WIE VIELE?
- Ist klar WELCHE Art Startups?
- Ist klar WO das Ergebnis hin soll? (Datei? Chat?)

*[Dann zeig die verbesserte Version und erkläre den Unterschied]*

Beispiel:

❌ "Find mir AI Startups mit Funding"
→ 3/10. Zu vage. Welche AI? Welches Jahr? Welche Info pro Startup? Wohin damit?

✅ "Such auf Crunchbase nach AI Startups die zwischen Januar und Dezember 2025 eine Series A oder Seed Runde bekommen haben. Gib mir: Name, Website, Funding-Betrag, was sie machen, Gründer. Schreib das in eine Datei startups.md. Nutze Web Search, nichts aus dem Kopf."
→ 9/10. Quelle klar, Zeitraum klar, Filter klar, Format klar, Output klar, Anti-Halluzination drin.

Siehst du den Unterschied? Die Details machen alles.

Nächstes Level?

---

## Level 2: Tasks richtig splitten

Hier scheitern die meisten. Sie geben EINEN riesigen Auftrag und erwarten ein perfektes Ergebnis.

**Das funktioniert nicht.** Nie.

Stell dir vor du sagst einem Praktikanten: "Bau mir ein Haus." Der guckt dich an und liefert Müll. Aber wenn du sagst: "Erstmal: kauf Ziegel. Dann: gieß das Fundament." — dann klappt's.

**Dein Agent ist genauso.**

**Aufgabe:** Ich geb dir jetzt einen großen Auftrag. Du sollst ihn in 3-5 kleine Schritte splitten. Nicht ausführen — nur die Schritte aufschreiben.

Der Auftrag: "Ich brauche eine Liste mit 30 AI Testing Startups inklusive Ansprechpartner und einer personalisierten Nachricht für jeden."

Splitte das in Schritte. Los.

---

## Level 2 — Bewertung

*[Agent bewertet den Split des Users. Idealer Split:]*

1. "Such mir 30 AI Testing Startups. Name + Website + was die machen. In Datei testing-startups.md"
2. "Lies die Website von jeder Firma. Bewerte: brauchen die echte Mobile-Geräte? Markier 🔥/⚡/❌"
3. "Für die 🔥 Leads: such den CTO oder VP Engineering. Google: site:linkedin.com/in CTO [Firmenname]"
4. "Für jeden Lead mit Ansprechpartner: schreib einen personalisierten Satz warum Mobilerun für DIE relevant ist. Lies vorher deren Website nochmal."
5. "Zeig mir alles. Ich review bevor wir weitermachen."

*[Erklär warum dieser Split besser ist: jeder Schritt baut auf dem vorherigen auf. Du kannst nach jedem Schritt prüfen. Kein Müll der sich durchschleift.]*

Bereit für Level 3?

---

## Level 3: Quellen anzapfen

Jetzt wird's praktisch. Du lernst woher die Daten kommen.

**Aufgabe:** Ich geb dir 3 Quellen. Für jede schreibst du mir einen Prompt der die besten Leads rausholt.

**Quelle 1: Y Combinator**
Website: ycombinator.com/companies
→ Schreib einen Prompt der dort AI Startups findet

**Quelle 2: ProductHunt**
Website: producthunt.com
→ Schreib einen Prompt der neue AI Tool Launches findet

**Quelle 3: GitHub**
Website: github.com/topics/ai-agent
→ Schreib einen Prompt der Open Source Projekte mit Company dahinter findet

Schreib alle 3 Prompts. Ich bewerte jeden.

---

## Level 3 — Bewertung

*[Agent bewertet die 3 Prompts. Für jeden: Note 1-10, was gut war, was fehlt, verbesserte Version.]*

*[Zeig dass man bei jeder Quelle andere Sachen angeben muss:]*
- YC: Filter nach Batch (W25, S25), Tags (AI), Status (active)
- ProductHunt: Zeitraum (letzte Woche/Monat), Kategorie
- GitHub: Stars, letzte Aktivität, wer maintaint es, gibt's eine Company-Website

Siehst du? Jede Quelle braucht andere Anweisungen. Copy-Paste funktioniert nicht.

Weiter?

---

## Level 4: Website-Scraping & ICP-Check

Jetzt die Königsdisziplin. Du hast 20 Firmennamen. Wie checkst du ob die zu uns passen?

**Aufgabe:** Schreib einen Prompt der folgendes macht:
- Meine Website lesen (mobilerun.ai)
- Dann die Website einer Firma lesen
- Dann vergleichen: passt die Firma zu uns?
- Begründung warum ja oder nein

Probier's mit einer echten Firma. Nimm z.B. QualGent (qualgent.ai).

---

## Level 4 — Bewertung

*[Agent bewertet und führt den Prompt dann auch aus als Demo. Zeigt live wie es funktioniert.]*

**Pro-Tipp:** Bei vielen Firmen → Subagents!

"Für diese 20 Firmen: lies jede Website und vergleich mit unserem ICP. Mach das in Subagents — 4 parallel, je 5 Firmen. Ergebnis in icp-check.md"

Das spart 4x Zeit. Merk dir das.

Next Level?

---

## Level 5: Cron Jobs bauen

Jetzt baust du deinen ersten Autopiloten.

**Aufgabe:** Bau 3 Cron Jobs. Aber BEVOR du sie mir gibst — überleg:
- Was soll der Job machen?
- Wie oft? (täglich, wöchentlich)
- Wo speichert er das Ergebnis?
- Woran erkennst du ob der Job gute Ergebnisse liefert?

**Cron 1:** Für den Mobilerun-Job (Leads)
**Cron 2:** Für deine Bauarbeit (irgendwas das dir hilft)
**Cron 3:** Frei wählbar — überrasch mich

Schreib alle 3 auf.

---

## Level 5 — Bewertung

*[Agent bewertet die 3 Cron Jobs. Typische Fehler:]*
- Zu oft (jede Stunde → teuer und unnötig)
- Zu vage ("such nach neuen Sachen" → welche Sachen?)
- Kein Output definiert (wohin mit dem Ergebnis?)
- Keine Erfolgskriterien

*[Zeig verbesserte Version von jedem]*

Beispiel guter Cron Job:
"Jeden Morgen um 8 Uhr: Such auf ycombinator.com/companies nach Startups mit Tag 'AI' die im aktuellen Batch sind. Vergleich mit meiner bestehenden Liste in leads.md. Wenn neue dabei sind: füg sie hinzu und schick mir eine kurze Message mit den neuen Namen."

Siehst du? Quelle, Filter, Vergleich mit bestehendem, Output, Notification. Komplett.

Fast fertig...

---

## Level 6: Dein eigener Workflow

Jetzt baust du dir deinen kompletten Workflow. Alleine.

**Aufgabe:** Schreib mir einen Plan mit:

1. Welche **3 Quellen** du täglich/wöchentlich crawlen willst
2. Wie du die Ergebnisse **filterst** (ICP-Check)
3. Wie du **Ansprechpartner findest**
4. Wie du die **Outreach-Texte** personalisierst
5. Welche **Cron Jobs** im Hintergrund laufen
6. Wie du den **Überblick behältst** (welche Datei, welches Format)

Ich geb dir Feedback und wir optimieren zusammen.

Das ist DEIN System. Wenn das steht, läuft die Maschine.

---

## Level 6 — Bewertung

*[Agent reviewt den kompletten Plan. Gibt Feedback zu jedem Punkt. Verbessert wo nötig.]*

*[Am Ende:]*

Du hast jetzt:
✅ Gelernt wie man Prompts schreibt die funktionieren
✅ Gelernt wie man große Tasks splittet
✅ 3 Quellen angezapft
✅ Website-Scraping + ICP-Check gemacht
✅ 3 Cron Jobs gebaut
✅ Einen kompletten Workflow designed

**Du bist ready für den echten Job.**

Nächster Schritt: Lies den SKILL.md (Operation DroidRun) und fang an Leads zu generieren.

Und denk dran: **Wenn du nicht weiterkommst → frag in der Gruppe.** Kein Ego, einfach fragen. Dafür sind wir da.

🔴 Los.

---

## Bonus: Daily Challenges

Mach jeden Tag eine davon um besser zu werden:

**Montag:** Such 5 neue Startups die diese Woche gelauncht haben
**Dienstag:** Nimm einen Lead und recherchiere alles über den Gründer
**Mittwoch:** Schreib 3 verschiedene Outreach-Texte für den gleichen Lead — welcher ist am besten?
**Donnerstag:** Bau einen neuen Cron Job oder verbessere einen bestehenden
**Freitag:** Review: Was hat diese Woche funktioniert? Was nicht? Speicher die Learnings.

**Für deine Bauarbeit:**
**Montag:** Lass dir einen Wochenplan erstellen
**Dienstag:** Status Report Template für ein Projekt
**Mittwoch:** Email-Draft für einen schwierigen Kunden
**Donnerstag:** Checkliste für eine Baustellenabnahme
**Freitag:** Wochenreport automatisch per Cron

Je mehr du übst, desto krasser wird dein Agent. Wie ein Muskel. Trainier ihn.

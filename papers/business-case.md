# SecondSwipe: Wertentdeckung in der Kreislaufwirtschaft
### Ein Geschäftsfall für wischbasierte Produktempfehlung im Second-Hand-Markt

> **Status:** Öffentliches Arbeitspapier · Version 1.0 · 2025
> **Kontext:** Dieses Dokument ist der ausführliche Geschäftsfall zu SecondSwipe. Es ergänzt die
> [öffentliche Pitch-Seite](../index.html) und begründet mit wissenschaftlicher Literatur, *warum* das
> Produkt funktioniert und welcher wirtschaftliche Fall dahintersteht.

---

## Abstract

Gebrauchtmarktplätze sind heute Suchmaschinen: Sie setzen voraus, dass die Nutzerin bereits weiss,
wonach sie sucht. Genau dort verlieren sie den grössten Teil ihres Werts – die Entdeckung des
Unbekannten. SecondSwipe kehrt die Interaktion um: Der Katalog kommt als wischbarer Strom zur Nutzerin,
eine Entscheidung pro Artikel. Jeder Wisch ist zugleich ein perfekt gelabeltes, binäres
Verhaltenssignal, das ein persönliches Empfehlungsmodell speist. Dieses Arbeitspapier begründet den
Geschäftsfall entlang dreier literaturgestützter Säulen: (1) der Psychologie der Wahlüberlastung,
(2) der Ökonomik der Kreislaufwirtschaft und (3) der Theorie des impliziten Feedbacks in
Empfehlungssystemen. Es zeigt, warum die bewusst *einfache* Interaktion – eine binäre Entscheidung
pro Karte – kein UX-Kompromiss ist, sondern der strategische Kern.

---

## 1. Problemstellung

Zweitmarktkataloge sind strukturell auf *Bekanntes* optimiert: Filter, Suchbegriffe und Kategorien
belohnen die Nutzerin, die bereits einen konkreten Begriff im Kopf hat. Der weitaus grössere Wert
liegt jedoch im *Unbekannten* – dem Einzelstück, das man nicht gesucht hat, aber sofort will. Dieser
Wert bleibt systematisch liegen, weil die Interaktion ihn nicht zugänglich macht.

Drei Beobachtungen verschärfen das Problem:

1. **Suchbasiertes Stöbern ist Arbeit.** Jede Suchanfrage ist eine kognitive Hypothese über den
   eigenen Geschmack. Wer keinen Begriff hat, hat keinen Einstieg.
2. **Relevante Treffer werden verdünnt.** In langen Ergebnislisten sinkt die Sichtbarkeit guter
   Treffer; die Wahrscheinlichkeit, das richtige Stück zu sehen, fällt mit der Position.
3. **Präferenzen werden erfragt statt beobachtet.** Klassische Marktplätze fragen nach Interessen
   (Kategorien, Suchagenten). Solche Selbstauskünfte sind notorisch unzuverlässig (siehe Abschnitt 4).

SecondSwipe adressiert alle drei Punkte mit einer einzigen Designentscheidung: *eine Karte, eine
Entscheidung, ein Wisch.*

---

## 2. Theoretische Grundlagen

### 2.1 Die Psychologie der Wahlüberlastung (Choice Overload)

Die klassische Studie von **Iyengar & Lepper (2000)** zeigte am Verkaufsstand für Konfitüre: Ein
grösseres Sortiment (24 Sorten) zog zwar mehr Interessenten an (60 % blieben stehen gegenüber 40 %
bei 6 Sorten), führte aber zu dramatisch weniger Käufen (3 % gegenüber 30 %). **Schwartz (2004)**
popularisierte diesen Befund als „Paradox of Choice": Mehr Optionen erhöhen die kognitive Last,
nicht die Zufriedenheit.

Die Forschung ist differenzierter, als die populäre Lesart nahelegt. Die Meta-Analyse von
**Scheibehenne, Greifeneder & Todd (2010)** zeigt, dass der Wahlüberlastungseffekt nicht universell
ist, sondern von Moderatoren abhängt – insbesondere davon, ob die Nutzerin klare Präferenzen hat,
wie die Optionen präsentiert werden und wie viel Aufwand die Bewertung erfordert. Die konzeptuelle
Übersicht von **Chernev, Böckenholt & Goodman (2015)** bestätigt diesen moderierenden Charakter.

Für SecondSwipe ist das die zentrale Lehre: **Nicht die Zahl der Optionen ist das Problem, sondern
die Gleichzeitigkeit der Bewertung.** Ein Wisch präsentiert genau eine Option zur Zeit und reduziert
die Entscheidung auf ein binäres Urteil. Das entschärft die Überlast nicht durch weniger Sortiment,
sondern durch eine *sequentielle* statt *simultane* Wahlarchitektur. Die Bewertungslast pro Moment
sinkt auf ihr Minimum: eine binäre, affektive Reaktion („Ja" / „Nein"), die Kahneman (2011) als
System-1-Urteil beschreibt – schnell, intuitiv, nahezu kostenlos.

### 2.2 Die Ökonomik der Kreislaufwirtschaft

Die Kreislaufwirtschaft ist kein Nischenphänomen, sondern ein etablierter Rahmen zur
Entkopplung von Wertschöpfung und Ressourcenverbrauch. **Kirchherr, Reike & Hekkert (2017)**
definieren sie anhand von 114 untersuchten Definitionen als ein System, das Abfall durch Reduktion,
Wiederverwendung und Recycling minimiert. **Geissdoerfer et al. (2017)** grenzen die Kreislaufwirtschaft
von Nachhaltigkeit ab und betonen den Wiederverwendungskreislauf (*reuse*) als Kernmechanismus.

Für den Wiederverkauf ist entscheidend: **Der Flaschenhals der Kreislaufwirtschaft ist nicht die
Bereitschaft, sondern die Auffindbarkeit.** Ein Artikel, der nicht wiederentdeckt wird, verlässt den
Kreislauf – unabhängig davon, wie nachhaltig er ursprünglich hergestellt wurde. Jede erfolgreiche
Wiederentdeckung eines gebrauchten Stücks substituiert potenziell eine Neuproduktion. SecondSwipe
positioniert sich damit nicht als „grünes" Produkt, sondern als **Infrastruktur für Wiederverwendung**:
Es senkt die Transaktionskosten der Wiederentdeckung und erhöht dadurch die Durchsatzrate des
Reuse-Kreislaufs. Die **Ellen MacArthur Foundation** beschreibt genau diese Rolle – Plattformen, die
Wiederverwendung skalierbar machen – als zentralen Hebel der Kreislaufwirtschaft.

### 2.3 Implizites Feedback und Empfehlungssysteme

Empfehlungssysteme klassifiziert die Literatur traditionell nach der Signalquelle. **Koren, Bell &
Volinsky (2009)** etablierten mit Matrix-Faktorisierung den Standard für *explizites* Feedback
(Bewertungen) und erweiterten ihn auf *implizite* Signale. **Hu, Koren & Volinsky (2008)** zeigten,
dass implizite Signale (Klicks, Käufe) dichter und ehrlicher sind als explizite Bewertungen, aber
eine sorgfältige Behandlung erfordern, weil sie Rauschen enthalten. Das Handbuch von
**Ricci, Rokach & Shapira (2011)** systematisiert diese Unterscheidung.

Ein Wisch ist ein Sonderfall des impliziten Feedbacks mit zwei bemerkenswerten Eigenschaften:

1. **Binär und damit eindeutig gelabelt.** Ein Wisch nach rechts ist ein positives, ein Wisch nach
   links ein negatives Label. Es gibt keine fünfstufige Skala mit individueller Kalibrierung – das
   Signal ist für jede Nutzerin identisch kodiert.
2. **Verhaltensbasiert statt selbstberichtet.** Der Wisch ist *gezeigtes* Verhalten. Die
   psychologische Literatur ist hier eindeutig: Selbstauskünfte über Präferenzen sind systematisch
   verzerrt (soziale Erwünschtheit, mangelnde Introspektion). Beobachtetes Verhalten umgeht diese
   Verzerrung.

Diese Eigenschaften machen jeden Wisch zu einem *perfekt gelabelten Trainingsbeispiel* – ein
ungewöhnlich sauberes Signal für ein Empfehlungssystem.

---

## 3. Das Produkt

SecondSwipe präsentiert Second-Hand-Artikel als Kartenstapel. Die Nutzerin wischt nach rechts
(„gefällt mir") oder links („nicht"). Aus den Wischern lernt ein persönliches Modell, das den Stapel
fortlaufend neu ordnet. Drei Merkmale definieren das Produkt:

- **Sequenzielle Entscheidung statt Suche.** Eine Option zur Zeit, eine binäre Reaktion. Kein
  Filterformular, keine Suchsyntax.
- **Transparentes Profil.** Die Nutzerin sieht, was das Modell über sie gelernt hat – als lesbare
  Präferenzen („Mid-Century stärker als Bauhaus") statt als Blackbox.
- **Privatsphäre durch Architektur.** Ein Modell pro Nutzerin, trainiert ausschliesslich auf den
  eigenen Wischern. Keine Cross-User-Verknüpfung, keine Tracker.

---

## 4. Warum die binäre Entscheidung der strategische Kern ist

Der Wisch ist kein UX-Detail, sondern die Geschäftslogik selbst. Die Argumentation:

1. **Niedrigste Bewertungskosten.** Die psychologische Literatur zeigt, dass die Bereitschaft zur
   Interaktion mit der kognitiven Last sinkt. Eine binäre, affektive Reaktion hat die geringstmögliche
   Last. Das maximiert die Zahl der Signale pro Nutzerin pro Sitzung.
2. **Saubere Labels.** Jede Nutzerin produziert dieselbe, unverzerrte Signalstruktur. Das ist die
   Voraussetzung dafür, dass ein Modell überhaupt verlässlich lernen kann – und ein Kontrast zu
   Selbstauskünften, die von Nutzerin zu Nutzerin anders kalibriert sind.
3. **Kalte Starts bleiben ehrlich.** Mit wenigen Wischern ist das Modell bewusst unsicher. SecondSwipe
   verschweigt das nicht, sondern zeigt es. Diese Ehrlichkeit ist ein Vertrauensmerkmal und
   unterscheidet das Produkt von Blackbox-Algorithmen, die Zuversicht simulieren.
4. **Ein kontinuierlicher Datenstrom.** Jeder Wisch ist ein neues Label. Das Modell verbessert sich
   mit jeder Interaktion, ohne dass die Nutzerin etwas „zusätzlich" tun müsste. Lernen ist ein
   Nebenprodukt der Nutzung.

---

## 5. Markt und Geschäftsmodell

Der Second-Hand-Markt wächst strukturell – getrieben von Nachhaltigkeitsbewusstsein, Preisvorteilen
und der Normalisierung von Wiederverkauf. Branchenberichte (u. a. der **Ellen MacArthur Foundation**
und die jährlichen Resale-Reports der Industrie) dokumentieren dieses Wachstum über Mode, Möbel und
Elektronik hinweg.

SecondSwipe monetarisiert nicht über Daten, sondern über die **Wiederverkaufs-Transaktion**, die seine
Empfehlung auslöst:

- **Vermittlungsprovision** auf verkaufte Artikel, die über die Merkliste (alle Rechts-Wischer) in
  einen Kauf münden.
- **Partner-/Affiliate-Modelle** mit bestehenden Marktplätzen, deren Katalog SecondSwipe erschliesst.

Der strategische Vorteil ist die **Empfehlungsqualität als Differenzierungsmerkmal**: Je besser die
Wiederentdeckung, desto höher die Konversion und desto wertvoller die Vermittlung. Die Datenmenge
(Wischer) ist zugleich der Lernstoff, der die Empfehlung weiter verbessert – ein positiver
Rückkopplungskreislauf, der mit jeder Nutzerin stärker wird.

---

## 6. Grenzen und Risiken

Ein universitär belastbarer Geschäftsfall benennt auch die Schwächen:

- **Kalter Start.** Die ersten Wischer sind nahezu zufällig; das Produkt braucht eine kritische Masse
  an Interaktionen, bevor die Empfehlung spürbar wird. (Siehe das [technische Papier](technical.md)
  für die quantitative Behandlung.)
- **Verweildauer als schwaches Signal.** Die Betrachtungszeit einer Karte ist ein nützlicher, aber
  verrauschter Indikator für Ambivalenz – sie wird nur als weiche Gewichtung eingesetzt.
- **Einzelmarktplatz-Abhängigkeit.** Als Vermittler ist SecondSwipe auf Katalogzugänge angewiesen;
  Exklusivität oder Datenzugang sind Verhandlungssache.
- **Skalierung.** Das bewusst einfache Modell ist für Tausende, nicht Millionen Nutzer ausgelegt.
  Der Übergang zu grösserer Architektur ist geplant, aber nicht trivial.

---

## 7. Fazit

SecondSwipe begründet seinen Geschäftsfall nicht durch mehr Features, sondern durch eine präzisere
Interaktion: die Reduktion der Produktbewertung auf ein einziges, binäres, beobachtbares Signal. Die
Literatur zur Wahlüberlastung erklärt, warum diese Reduktion die Bewertungslast senkt; die
Kreislaufwirtschaftsliteratur erklärt, warum Wiederentdeckung der entscheidende Engpass ist; die
Empfehlungssystemliteratur erklärt, warum ein binäres Verhaltenssignal ein ungewöhnlich sauberes
Lernsignal ist. Gemeinsam begründen diese drei Stränge, warum SecondSwipe nicht nur ein weiterer
Marktplatz ist, sondern eine andere Art, Second-Hand zu entdecken.

---

## Referenzen

- Chernev, A., Böckenholt, U., & Goodman, J. (2015). Choice overload: A conceptual review and
  meta-analysis. *Journal of Consumer Psychology, 25*(2), 333–358.
- Geissdoerfer, M., Savaget, P., Bocken, N. M. P., & Hultink, E. J. (2017). The Circular Economy –
  A new sustainability paradigm? *Journal of Cleaner Production, 143*, 757–768.
- Hu, Y., Koren, Y., & Volinsky, C. (2008). Collaborative filtering for implicit feedback datasets.
  *Proceedings of the IEEE International Conference on Data Mining (ICDM)*.
- Iyengar, S. S., & Lepper, M. R. (2000). When choice is demotivating: Can one desire too much of a
  good thing? *Journal of Personality and Social Psychology, 79*(6), 995–1006.
- Kahneman, D. (2011). *Thinking, Fast and Slow*. Farrar, Straus and Giroux.
- Kirchherr, J., Reike, D., & Hekkert, M. (2017). Conceptualizing the circular economy: An analysis
  of 114 definitions. *Resources, Conservation and Recycling, 127*, 221–232.
- Koren, Y., Bell, R., & Volinsky, C. (2009). Matrix factorization techniques for recommender
  systems. *Computer, 42*(8), 30–37.
- Ricci, F., Rokach, L., & Shapira, B. (2011). *Recommender Systems Handbook*. Springer.
- Scheibehenne, B., Greifeneder, R., & Todd, P. M. (2010). Can there ever be too many options? A
  meta-analytic review of choice overload. *Journal of Consumer Research, 37*(3), 409–425.
- Schwartz, B. (2004). *The Paradox of Choice: Why More Is Less*. Ecco.
- Ellen MacArthur Foundation. *Circular Economy Reports*. (Laufende Publikationsreihe.)

---

*Dieses Dokument ist Teil der öffentlichen Dokumentation von SecondSwipe. Das begleitende
[technische Papier](technical.md) behandelt Modellwahl und Systemarchitektur.*

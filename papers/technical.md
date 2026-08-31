# SecondSwipe: Bayes'sche Empfehlung aus spärlichem, binärem Feedback
### Technisches Arbeitspapier zu Modellwahl, Inferenz und Systemarchitektur

> **Status:** Öffentliches Arbeitspapier · Version 1.0 · 2025
> **Kontext:** Dieses Dokument begründet die technischen Entscheidungen hinter SecondSwipe. Es
> ergänzt die [öffentliche Pitch-Seite](../index.html) und das
> [Geschäftsfall-Papier](business-case.md).

---

## Abstract

SecondSwipe empfiehlt Second-Hand-Artikel aus einem Strom binärer Wisch-Entscheidungen. Jede
Nutzerin besitzt ein eigenes Modell, das ausschliesslich auf ihren eigenen, binären
Verhaltenssignalen trainiert wird. Wir begründen hier die zentralen Entscheidungen: (1) **VLM-basierte
Feature-Extraktion** aus Produktbildern zur automatischen Gewinnung interpretierbarer Merkmale,
(2) eine **logistische Regression mit Bayes'schem Posterior** statt kollaborativem Filtern oder Deep
Learning, (3) eine **Laplace-Approximation** als analytische Inferenzmethode, die in einer einzigen
Request laufen kann, und (4) **Thompson Sampling** als explorationsfreier Bandit für die
Kartenauswahl. Wir zeigen, warum diese Kombination die Anforderungen – Kaltstart-Ehrlichkeit,
Interpretierbarkeit, Unsicherheitsquantifizierung und Privatsphäre – bei minimaler Systemkomplexität
erfüllt, und wie implizite Präferenzen aus reinem Wischverhalten emergieren, die der Nutzer selbst
nicht artikulieren könnte.

---

## 1. Problemstellung und Anforderungen

Das Empfehlungsproblem von SecondSwipe ist durch vier Eigenschaften geprägt:

1. **Binäres Feedback.** Jeder Wisch ist ein Label `y ∈ {0, 1}` (rechts = 1, links = 0).
2. **Spärlicher Kaltstart.** Die Nutzerin beginnt mit null Signalen; das Modell muss bereits nach
   wenigen Interaktionen funktionieren und dabei seine Unsicherheit ehrlich ausweisen.
3. **Interpretierbarkeit.** Die Nutzerin soll ihr gelerntes Profil *lesen* können. Eine Blackbox
   ist ausgeschlossen.
4. **Privatsphäre durch Architektur.** Ein Modell pro Nutzerin, keine Cross-User-Verknüpfung, keine
   Tracker. Das verbietet Verfahren, die von Natur aus Nutzerübergreifende Daten benötigen.

Aus diesen Anforderungen folgt die Architektur unmittelbar. Wir stellen sie in Abschnitt 8 dar.

---

## 2. Modellwahl: Warum logistische Regression?

Die Modellfamilie muss drei Bedingungen gleichzeitig erfüllen: **interpretierbar**, **bei wenig
Daten stabil** und **natürlich kalibriert**.

### 2.1 Verworfen: Kollaboratives Filtern / Matrix-Faktorisierung

Matrix-Faktorisierung (**Koren, Bell & Volinsky, 2009**) ist der Standard für Empfehlungssysteme,
verletzt aber gleich zwei Anforderungen:

- **Cross-User-Daten sind konstitutiv.** Die latente Nutzer- und Artikelmatrix entsteht aus den
  Interaktionen *aller* Nutzer. Das ist mit „ein Modell pro Nutzerin, keine Verknüpfung" unvereinbar.
- **Kaltstart.** Neue Nutzerinnen haben keine Zeile in der Matrix; neue Artikel keine Spalte. Beides
  erfordert Zusatzheuristiken.

Für SecondSwipe ist Kollaboratives Filtern daher strukturell ausgeschlossen – nicht aus
Performance-Gründen, sondern weil es die Privatsphäre- und Kaltstart-Anforderungen verletzt.

### 2.2 Verworfen: Deep Learning

Neuronale Empfehler (**vgl. Covington, Adams & Sargin, 2016** für YouTube) liefern hohe
Vorhersagekraft, aber zu einem Preis, den SecondSwipe nicht zahlen will:

- **Datenhunger.** Tiefe Modelle benötigen typischerweise Grössenordnungen mehr Interaktionen, als
  eine einzelne Nutzerin je produziert. Bei einem Nutzer-lokalen Modell ist das unerreichbar.
- **Intransparenz.** Latente Embeddings sind nicht „lesbar". Das Profil-Feature wäre unmöglich.
- **Schlechte Kalibrierung.** **Guo, Pleiss, Sun & Weinberger (2017)** dokumentierten, dass moderne
  tiefe Netze oft überkonfident sind – ihre Konfidenz stimmt nicht mit der empirischen Trefferquote
  überein. Für ein Produkt, dessen Versprechen *ehrliche* Unsicherheit ist, ist das disqualifizierend.

### 2.3 Gewählt: Logistische Regression mit Bayes'schem Posterior

Die logistische Regression (**Bishop, 2006**) erfüllt alle drei Bedingungen:

- **Interpretierbar.** Jedes Gewicht entspricht einer lesbaren Präferenz („Holz > Metall",
  „Mid-Century > Bauhaus"). Die Gewichte *sind* das Profil.
- **Datenökonomisch.** Ein lineares Modell mit regulierendem Prior lernt auch aus wenigen Dutzend
  Beobachtungen sinnvolle (wenn auch unsichere) Parameter.
- **Natürlich kalibriert.** Die Sigmoid-Ausgabe ist per Konstruktion eine Wahrscheinlichkeit; mit
  einem Posterior über die Gewichte ergibt sich eine volle prädiktive Verteilung.

Die Modellierung: Artikelmerkmale `x` (Kategorie, Ära, Material, Farbe, Preisband) werden mit
Gewichten `w` linear kombiniert und durch die Sigmoid-Funktion in eine Like-Wahrscheinlichkeit
überführt:

```
p(like = 1 | x, w) = σ(w·x) = 1 / (1 + exp(-w·x))
```

Der entscheidende Schritt ist, `w` nicht als Punktschätzung zu behandeln, sondern als
**Zufallsvariable mit Posterior-Verteilung**. Das liefert Unsicherheit, Regularisierung und
Exploration aus einer einzigen Konstruktion.

---

## 3. Feature-Extraktion und die Emergenz impliziter Präferenzen

Das Modell benötigt strukturierte Merkmale `x`, um aus einem Wisch lernen zu können. SecondSwipe
gewinnt diese Merkmale nicht durch manuelle Kategorisierung oder Nutzerbefragung, sondern durch
automatisierte Analyse der Produktbilder und -beschreibungen.

### 3.1 VLM-basierte Attributextraktion

Jeder Artikel im Katalog wird vor der ersten Präsentation durch ein **Vision-Language Model (VLM)**
verarbeitet. Das VLM extrahiert aus Bild und Beschreibung eine Reihe semantischer Attribute:

- **Material:** Holz, Metall, Kunststoff, Stoff, Leder, Stein, Glas, etc.
- **Farbe:** Dominante Farbtöne und Farbtemperaturen.
- **Stil/Ära:** Mid-Century Modern, Bauhaus, Industrial, Skandinavisch, Vintage, etc.
- **Kategorie:** Sofa, Leuchte, Stuhl, Tisch, Regal, etc.
- **Zustand:** Neu, sehr gut, gebraucht, restaurierungsbedürftig.
- **Preisband:** Ordinale Einordnung des Preises.

Die Extraktion basiert auf Architekturen wie **CLIP** (**Radford et al., 2021**), die visuelle und
textuelle Repräsentationen in einem gemeinsamen Embedding-Raum abbilden. Durch Prompting oder
fine-tuning auf Second-Hand-Domänen werden die rohen Embeddings in diskrete, interpretierbare
Labels überführt. Diese Labels bilden den Merkmalsvektor `x` für das logistische Modell.

Der Vorteil dieser Pipeline ist die **Skalierbarkeit ohne manuelle Annotation**: Jeder neue Artikel
im Katalog erhält automatisch seine Merkmale, ohne dass ein Mensch ihn kategorisieren muss.

### 3.2 Der Gewichtsvektor als Wichtigkeitsmatrix

Nach dem Training enthält der Posterior über die Gewichte `w` die gesamte Information über die
Präferenzen der Nutzerin. Der Erwartungswert `E[w]` ist dabei eine **Wichtigkeitsmatrix**:

- **|w_i| (Absolutwert):** Wie wichtig ist Merkmal *i* für die Entscheidung? Ein grosses |w_i|
  bedeutet, dass dieses Merkmal die Like-Wahrscheinlichkeit stark beeinflusst.
- **sign(w_i) (Vorzeichen):** In welche Richtung wirkt das Merkmal? Positiv = präferenzsteigernd,
  negativ = präferenzmindernd.

Beispiel: Wenn `w_Holz = +0.8` und `w_Kunststoff = -0.6`, dann erhöht Holz die Like-Wahrscheinlichkeit
deutlich, während Kunststoff sie senkt. Die Differenz `|w_Holz| + |w_Kunststoff|` zeigt, dass Material
ein hochrelevantes Merkmal für diese Nutzerin ist.

Diese Matrix ist **nicht vorgegeben**, sondern emergiert ausschliesslich aus dem Wischverhalten. Sie
ist das quantitative Abbild des individuellen Geschmacks.

### 3.3 Revealed Preference: Was Verhalten offenbart, was Introspektion verschweigt

Die zentrale wissenschaftliche Einsicht hinter SecondSwipe ist die **Revealed-Preference-Theorie**
(**Samuelson, 1938**): Präferenzen werden nicht durch Befragung ermittelt, sondern durch beobachtetes
Verhalten offenbart. Ein Nutzer kann vielleicht nicht artikulieren, dass er Holz gegenüber Kunststoff
präferiert – aber sein konsistentes Rechts-Wischen bei Holzartikeln und Links-Wischen bei
Kunststoffartikeln enthüllt diese Präferenz mit statistischer Sicherheit.

Dies steht im Kontrast zur **Stated Preference** (selbstberichtete Präferenz), die in der
psychologischen Literatur als systematisch verzerrt gilt:

- **Soziale Erwünschtheit:** Nutzer geben Antworten, die sozial akzeptabel erscheinen, nicht ihre
  wahren Präferenzen.
- **Mangelnde Introspektion:** Viele Präferenzen sind implizit und der bewussten Reflexion nicht
  zugänglich (**Greenwald & Banaji, 1995**).
- **Attitude-Behavior Gap:** Selbst wenn Nutzer ihre Präferenz korrekt berichten, korreliert sie
  oft schwach mit tatsächlichem Verhalten (**Ajzen & Fishbein, 1977**; **Nosek et al., 2007**).

SecondSwipe umgeht alle drei Verzerrungen, indem es **nur Verhalten misst**. Der Wisch ist ein
binärer, unverfälschter Akt der Wahl. Aus hunderten solcher Akte emergiert ein Präferenzprofil, das
genauer ist als jede Befragung – weil es auf gezeigtem, nicht auf behauptetem Geschmack basiert.

Ein konkretes Beispiel: Eine Nutzerin wischt konsequent rechts bei Artikeln aus Holz oder Stein und
links bei Kunststoff. Würde man sie fragen „Was ist dein Lieblingsmaterial?", könnte sie keine klare
Antwort geben – die Präferenz ist ihr nicht bewusst. Aber das Modell lernt sie trotzdem, weil das
Verhalten sie verrät. Genau diese **implizite Präferenzextraktion** ist der Kernwert von SecondSwipe.

---

## 4. Bayesianische Inferenz: Warum ein Posterior, warum Laplace?

### 4.1 Warum Bayes'sch statt Maximum-Likelihood?

Eine reine Maximum-Likelihood-Schätzung (MLE) liefert einen Punktwert, aber keine Aussage über
Unsicherheit – und sie ist bei spärlichen Daten instabil (Überanpassung an wenige Wischer). Ein
Bayes'scher Ansatz (**Gelman et al., 2013**) stellt einen Prior über `w` auf und kombiniert ihn mit
den Daten zum Posterior:

```
Posterior(w | D) ∝ Likelihood(D | w) · Prior(w)
```

Der Prior wirkt als **Regularisierer**: Bei null oder wenigen Wischern dominiert er und hält die
Schätzung konservativ. Das ist exakt das Verhalten, das ein ehrlicher Kaltstart braucht.

### 4.2 Warum die Laplace-Approximation?

Vollständige Bayes'sche Inferenz über MCMC (Markov-Chain-Monte-Carlo) ist für ein Modell, das *bei
jeder Empfehlungs-Request* neu trainiert wird, zu langsam. Die Laplace-Approximation
(**MacKay, 1992; Bishop, 2006**) bietet den entscheidenden Kompromiss: Sie approximiert den
Posterior durch eine multivariate Normalverteilung um das Posterior-Maximum `w_MAP`:

```
Posterior(w | D) ≈ N( w_MAP, H⁻¹ )
```

Dabei ist `H` die negative Hesse-Matrix der log-Posterior-Dichte an der Stelle `w_MAP`. Ein
Newton-Schritt liefert zugleich Punktwert und Kovarianzmatrix – und damit pro Vorhersage ein
Unsicherheitsmass. Bei logistischer Regression (log-konkaver log-Posterior) ist die
Gauss-Approximation nahe dem Modus eine gute Näherung.

Die Wahl ist also pragmatisch und begründet: **MCMC wäre genauer, aber für Echtzeit-Inferenz zu
teuer; Laplace liefert Unsicherheit in Millisekunden und ist für die Dimensionen eines
Nutzer-lokalen Modells ausreichend.**

---

## 5. Exploration: Warum Thompson Sampling?

Ein Empfehler, der immer nur den erwarteten Bestwert zeigt, erkundet nie und verharrt in einer
Filterblase. Die klassische Lösung ist Exploration. Wir wählen **Thompson Sampling**
(**Thompson, 1933**) – posterior-samples der Gewichte:

```
for candidate in catalog:
    w̃ ~ N(w_MAP, H⁻¹)          # eine Stichprobe aus dem Posterior
    score = σ(w̃ · x)            # Bewertung mit dem gezogenen Gewicht
choose argmax(score)
```

Die Begründung liegt in der Literatur:

- **Chapelle & Li (2011)** zeigten empirisch, dass Thompson Sampling im Bandit-Setting klassische
  Verfahren (ε-greedy, UCB) oft übertrifft und deutlich robuster als ε-greedy ist, weil es keine
  manuell zu wählende Explorationsrate braucht.
- **Russo et al. (2018)** geben in ihrem Tutorial die theoretische Fundierung: Thompson Sampling
  ist „probability matching" – es zieht jede Aktion mit der Wahrscheinlichkeit, dass sie optimal ist.
  Exploration entsteht automatisch aus der Posterior-Unsicherheit.
- **Kaltstart-kompatibel.** Bei breitem Posterior (wenig Daten) sind die Stichproben stark gestreut
  → viel Exploration. Mit wachsender Datenlage schrumpft der Posterior → das Sampling konvergiert
  gegen reine Exploitation. Es gibt keinen einzigen handzutunenden Parameter.

Damit ist Thompson Sampling die natürliche Wahl: **Es nutzt genau die Posterior-Verteilung, die wir
ohnehin berechnen, und regelt Exploration/Exploitation ohne Stellschrauben.**

---

## 6. Unsicherheit und Kalibrierung

Die Unsicherheit einer Vorhersage folgt aus der Kovarianzmatrix über die lineare Propagierung:

```
var(p) = (∂p/∂w)ᵀ · H⁻¹ · (∂p/∂w)
ci₉₅  = p ± 1.96 · √var(p)
```

Wichtig ist die **ehrliche Interpretation**: Bei wenigen Wischern ist dieses Intervall breit (nahe
`[0.3, 0.9]`), nicht künstlich eng. Kalibrierung wird gemessen, nicht behauptet:

- **Brier-Score** (**Brier, 1950**) als Gesamtmass der probabilistischen Güte.
- **Kalibrierungskurven** (reliability diagrams) nach **Niculescu-Mizil & Caruana (2005)**, um zu
  prüfen, ob „78 %" auch empirisch 78 % Treffer bedeutet.
- **AUC / ROC** als diskriminatives Mass (trennt das Modell Likes von Nicht-Likes?).

Diese Metriken erscheinen im Analyse-Tab als Teil des transparenten Profils.

---

## 7. Verweildauer als weiches Signal

Neben dem binären Label erfasst SecondSwipe die **Betrachtungsdauer** einer Karte (zwischen Anzeige
und Wisch). Sie ist ein Indikator für Ambivalenz: langes Zögern vor einem „Ja" deutet auf relevante
Unsicherheit. Die Dauer wird als **weiche Gewichtung** in den Posterior eingebracht, nicht als
hartes Label:

```
score = σ(w·x) · (1 + λ · similarity(dwellCard, candidate))
```

`λ` ist klein und gedeckelt. Die Begründung für die Zurückhaltung ist empirisch: Verweildauer ist
ein **lautes Signal** (Ablenkung, Ladezeiten, unentschlossenes Betrachten ohne Kaufabsicht). Die
Literatur zum impliziten Feedback (**Hu, Koren & Volinsky, 2008**) mahnt genau diese Vorsicht an –
implizite Signale sind dicht, aber verrauscht und dürfen das harte Label nicht übersteuern.

---

## 8. Systemarchitektur und die Wahl des Weges

```
Browser (React/Vite)
    │  HTTP
    ▼
Express (Node.js, TypeScript)
    ├─ /api/recommend  → Modell laden, Laplace, Thompson-Sample, Top-K
    ├─ /api/swipe      → Wisch speichern, Posterior aktualisieren
    ├─ /api/profile    → Gewichte, Kalibrierung, Reset
    └─ SQLite (ein Modell + Wischer pro Nutzerin)
```

Die Architekturentscheidungen folgen demselben Prinzip wie die Modellwahl: **minimale Komplexität
bei erfüllten Anforderungen.**

- **Ein Prozess, eine Maschine.** Bei Pilot-Grösse ist das Modell pro Request aus den SQLite-Zeilen
  der Nutzerin aufgebaut und in Millisekunden gefittet. Kein separater Inferenzdienst, keine
  Serialisierung des Modells über Prozessgrenzen.
- **TypeScript für das ML.** Das Modell ist reine Mathematik (Sigmoid, Newton-Schritt, Hesse-Matrix,
  Matrixinversion). Eine zweite Sprache (Python) und ein zweiter Dienst würden Laufzeit und
  Deployment-Komplexität erhöhen, ohne dass ein Pilot sie braucht.
- **SQLite statt Postgres.** Ein Einzelprozess mit einer Schreiberin pro Nutzerin – SQLite deckt das
  vollständig ab und entfällt als Betriebskomponente. Der Pfad zu Postgres ist für die Skalierung
  vorgezeichnet.
- **Keine GPU.** Der Rechenaufwand ist lächerlich gering; die bewusste Abwesenheit einer
  Deep-Learning-Pipeline ist ein Feature, kein Mangel.

Diese Route ist ein **Pilot-Kompromiss**, kein Skalierungsversprechen. Abschnitt 10 benennt die
Übergänge.

---

## 9. Evaluierungsmethodik

Ein wissenschaftlich belastbarer Empfehler wird an drei Achsen gemessen:

1. **Diskrimination (AUC).** Trennt das Modell gelikte von nicht-gelikten Artikeln?
2. **Kalibrierung (Brier, Reliability Diagram).** Stimmen die ausgegebenen Wahrscheinlichkeiten mit
   den beobachteten Frequenzen überein?
3. **Online-Regret.** Im Vergleich zu einer Zufalls-Baseline: Wie viele zusätzliche Likes erzeugt
   das Thompson-Sampling-Modell pro Sitzung? Regret ist das Mass für die Exploration/Exploitation-Güte.

Die ehrliche Kaltstart-Erkenntnis aus Abschnitt 6 bleibt dabei sichtbar: **Mit sehr wenigen Wischern
ist die Punktvorhersage nahezu zufällig, und das Produkt sagt das auch.**

---

## 10. Grenzen und nächste Schritte

- **Laplace ist eine Gauss-Näherung.** Sie ist bei logistischer Regression gut, aber nicht exakt.
  Für höhere Ansprüche ist variational inference (VI) oder MCMC der nächste Schritt – beides gegen
  Echtzeit-Budget abzuwägen.
- **Kaltstart ohne Cross-User-Wissen.** Der bewusste Verzicht auf kollaboratives Filtern begrenzt
  die Empfehlungsgüte in den ersten Sitzungen. Ein datenschutzkonformer Mittelweg (z. B.
  föderiertes Lernen oder vorab trainierte, nutzerunabhängige Merkmals-Priors) ist die naheliegende
  Erweiterung, ohne das „0 Tracker"-Versprechen zu brechen.
- **Skalierung.** Laplace + SQLite sind für Tausende, nicht Millionen Nutzer gedacht. Der
  definierte Pfad: Postgres für die Persistenz, ein separater Inferenzdienst für den Durchsatz,
  ggf. VI für strengere Posterior-Approximation.
- **Verweildauer.** Aktuell eine gedeckelte weiche Gewichtung. Eine systematische Evaluation ihres
  Vorhersagewerts ist offen.

---

## Referenzen

- Bishop, C. M. (2006). *Pattern Recognition and Machine Learning*. Springer.
- Brier, G. W. (1950). Verification of forecasts expressed in terms of probability.
  *Monthly Weather Review, 78*(1), 1–3.
- Chapelle, O., & Li, L. (2011). An empirical evaluation of Thompson sampling.
  *Advances in Neural Information Processing Systems (NeurIPS), 24*.
- Covington, P., Adams, J., & Sargin, E. (2016). Deep neural networks for YouTube recommendations.
  *Proceedings of the 10th ACM Conference on Recommender Systems (RecSys)*.
- Gelman, A., Carlin, J. B., Stern, H. S., Dunson, D. B., Vehtari, A., & Rubin, D. B. (2013).
  *Bayesian Data Analysis* (3rd ed.). CRC Press.
- Guo, C., Pleiss, G., Sun, Y., & Weinberger, K. Q. (2017). On calibration of modern neural
  networks. *Proceedings of the 34th International Conference on Machine Learning (ICML)*.
- Greenwald, A. G., & Banaji, M. R. (1995). Implicit social cognition: Attitudes, self-esteem, and
  stereotypes. *Psychological Review, 102*(1), 4–27.
- Hu, Y., Koren, Y., & Volinsky, C. (2008). Collaborative filtering for implicit feedback datasets.
  *Proceedings of the IEEE International Conference on Data Mining (ICDM)*.
- Koren, Y., Bell, R., & Volinsky, C. (2009). Matrix factorization techniques for recommender
  systems. *Computer, 42*(8), 30–37.
- MacKay, D. J. C. (1992). Bayesian interpolation. *Neural Computation, 4*(3), 415–447.
- Niculescu-Mizil, A., & Caruana, R. (2005). Predicting good probabilities with supervised
  learning. *Proceedings of the 22nd International Conference on Machine Learning (ICML)*.
- Nosek, B. A., et al. (2007). Pervasiveness and correlates of implicit attitudes and stereotypes.
  *European Review of Social Psychology, 18*(1), 36–88.
- Radford, A., Kim, J. W., Hallacy, C., Ramesh, A., Agarwal, G., Dhariwal, S., & Sutskever, I.
  (2021). Learning transferable visual models from natural language supervision (CLIP).
  *Proceedings of the 38th International Conference on Machine Learning (ICML)*.
- Russo, D., Van Roy, B., Kazerouni, A., Osband, I., & Wen, Z. (2018). A tutorial on Thompson
  sampling. *Foundations and Trends in Machine Learning, 11*(1), 1–96.
- Samuelson, P. A. (1938). A note on the pure theory of consumer's behaviour. *Economica, 5*(18),
  199–205.
- Thompson, W. R. (1933). On the likelihood that one unknown probability exceeds another in view of
  the evidence of two samples. *Biometrika, 25*(3–4), 285–294.

---

*Dieses Dokument ist Teil der öffentlichen Dokumentation von SecondSwipe. Das begleitende
[Geschäftsfall-Papier](business-case.md) behandelt die wirtschaftliche und psychologische Begründung.*

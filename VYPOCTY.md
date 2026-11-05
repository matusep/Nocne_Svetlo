# Výpočty: nočné svetlo s ranným budíkom

## Vstupné hodnoty

| Označenie | Hodnota |
|---|---|
| Batéria $V_{bat}$ | 9 V |
| Úbytok na D2 (SS14) $V_{D2}$ | 0,3 V (odhad) |
| Rf | 33 kΩ |
| Rl | 390 Ω |
| Rb1 | 10 kΩ |
| Rb2 | 47 kΩ |
| Rc2 | 10 kΩ |
| Rp | 100 Ω |
| U3: RT3A, RT3B, CT3 | 10 kΩ, 47 kΩ, 4,7 µF |
| U2: RT2A, RT2B, CT2 | 4,7 kΩ, 22 kΩ, 10 nF |
| LDR1 (GT36528) | 8 – 20 kΩ pri 10 lux, $\gamma = 0{,}6$ |
| LED: úbytok $V_f$ | 2,8 V (odhad) |
| Tranzistor: $V_{CE(sat)}$ | 0,1 V |
| NE555: výstup HIGH | $V_{CC} - 1{,}7$ V |

---

## 1. Napájacie napätie

**Čo počítame:** napätie, ktoré dostanú obvody po prechode cez ochrannú diódu D2.

$$V_{CC} = V_{bat} - V_{D2} = 9 - 0{,}3 = 8{,}7\ \mathrm{V}$$

**Výsledok:** $V_{CC} = 8{,}7\ \mathrm{V}$

---

## 2. Delič napätia LDR1 + Rf

**Čo počítame:** napätie v bode X (vstup U1) v závislosti od odporu LDR1.

$$X = V_{CC} \cdot \frac{R_f}{R_f + R_{LDR}}$$

---

## 3. Prahy Schmittovho spúšťača U1

**Čo počítame:** pri akom odpore LDR1 sa U1 prepne. NE555 prepína pri $\tfrac{1}{3}V_{CC}$ a $\tfrac{2}{3}V_{CC}$.

**Horný prah** (svetlo, lampa sa vypne):

$$\frac{R_f}{R_f + R_{LDR}} = \frac{2}{3} \;\Rightarrow\; R_{LDR} = \frac{R_f}{2} = \frac{33}{2} = 16{,}5\ \mathrm{k\Omega}$$

**Dolný prah** (tma, lampa sa zapne):

$$\frac{R_f}{R_f + R_{LDR}} = \frac{1}{3} \;\Rightarrow\; R_{LDR} = 2R_f = 2 \cdot 33 = 66\ \mathrm{k\Omega}$$

**Kontrola** pri $V_{CC} = 8{,}7\ \mathrm{V}$:

$$R_{LDR} = 16{,}5\ \mathrm{k\Omega}:\quad X = 8{,}7 \cdot \frac{33}{33 + 16{,}5} = 5{,}8\ \mathrm{V} = \tfrac{2}{3}\cdot 8{,}7\ \mathrm{V}$$

$$R_{LDR} = 66\ \mathrm{k\Omega}:\quad X = 8{,}7 \cdot \frac{33}{33 + 66} = 2{,}9\ \mathrm{V} = \tfrac{1}{3}\cdot 8{,}7\ \mathrm{V}$$

**Výsledok:** zapnutie lampy pri $R_{LDR} = 66\ \mathrm{k\Omega}$, vypnutie pri $R_{LDR} = 16{,}5\ \mathrm{k\Omega}$.

---

## 4. Osvetlenie, pri ktorom sa lampa prepne

**Čo počítame:** pri akom osvetlení $E$ (v luxoch) má LDR1 odpor 66 kΩ a 16,5 kΩ.

$$R = R_{10}\left(\frac{10}{E}\right)^{\gamma} \;\Rightarrow\; E = 10\left(\frac{R_{10}}{R}\right)^{1/\gamma}$$

Príklad (zapnutie lampy, $R = 66\ \mathrm{k\Omega}$, $R_{10} = 8\ \mathrm{k\Omega}$):

$$E = 10 \cdot \left(\frac{8}{66}\right)^{1/0{,}6} = 0{,}30\ \mathrm{lux}$$

| Prepnutie | $R_{LDR}$ | $E$ pri $R_{10} = 8\ \mathrm{k\Omega}$ | $E$ pri $R_{10} = 20\ \mathrm{k\Omega}$ |
|---|---|---|---|
| Lampa sa zapne | 66 kΩ | 0,30 lux | 1,4 lux |
| Lampa sa vypne | 16,5 kΩ | 3,0 lux | 13,8 lux |

**Výsledok:** zapnutie pri 0,3 – 1,4 lux, vypnutie pri 3 – 14 lux (podľa konkrétneho kusu LDR).

---

## 5. Prúd LED

**Čo počítame:** prúd cez LED D1 (nastavuje ho Rl).

$$I_{LED} = \frac{V_{CC} - V_f - V_{CE(sat)}}{R_l} = \frac{8{,}7 - 2{,}8 - 0{,}1}{390} = 14{,}9\ \mathrm{mA}$$

**Výsledok:** $I_{LED} = 14{,}9\ \mathrm{mA}$

---

## 6. Q1 je v saturácii

**Čo počítame:** či prúd bázy Q1 stačí na úplné otvorenie tranzistora.

Prúd bázy (výstup U1 v HIGH je $8{,}7 - 1{,}7 = 7{,}0\ \mathrm{V}$, $U_{BE} = 0{,}7\ \mathrm{V}$):

$$I_{B1} = \frac{7{,}0 - 0{,}7}{R_{b1}} = \frac{6{,}3}{10\ \mathrm{k\Omega}} = 0{,}63\ \mathrm{mA}$$

Nútené zosilnenie:

$$\beta_{\text{nútené}} = \frac{I_{LED}}{I_{B1}} = \frac{14{,}9}{0{,}63} \approx 24$$

**Výsledok:** $\beta_{\text{nútené}} = 24 < h_{FE}$ (2N3904 má typicky $\geq 100$), takže je Q1 v saturácii.

---

## 7. Výkon na odpore Rl

**Čo počítame:** či odpor Rl (puzdro 0603, 100 mW) nie je preťažený.

$$P_{R_l} = I_{LED}^{2} \cdot R_l = (0{,}0149)^2 \cdot 390 = 86\ \mathrm{mW}$$

**Výsledok:** 86 mW z 100 mW (v limite, malá rezerva).

---

## 8. Q2 je v saturácii (invertor)

**Čo počítame:** či Q2 stiahne bod DEN na nulu, keď je U1 v HIGH.

$$I_{B2} = \frac{7{,}0 - 0{,}7}{R_{b2}} = \frac{6{,}3}{47\ \mathrm{k\Omega}} = 0{,}134\ \mathrm{mA}$$

$$I_{C2} = \frac{V_{CC} - V_{CE(sat)}}{R_{c2}} = \frac{8{,}7 - 0{,}1}{10\ \mathrm{k\Omega}} = 0{,}86\ \mathrm{mA}$$

$$\beta_{\text{nútené}} = \frac{I_{C2}}{I_{B2}} = \frac{0{,}86}{0{,}134} \approx 6{,}4$$

**Výsledok:** $\beta_{\text{nútené}} = 6{,}4 \ll h_{FE}$, takže je Q2 v saturácii. DEN ≈ 0,1 V (pod hranicou resetu NE555, 0,4 V), takže U3 je vypnutý. Keď je Q2 zatvorený, Rc2 vytiahne DEN na $V_{CC}$ a U3 beží.

---

## 9. Pomalý oscilátor U3 (rytmus)

**Čo počítame:** frekvenciu pípania. Astabilný režim NE555, $R_A = 10\ \mathrm{k\Omega}$, $R_B = 47\ \mathrm{k\Omega}$, $C = 4{,}7\ \mathrm{\mu F}$.

$$t_H = 0{,}693\,(R_A + R_B)\,C = 0{,}693 \cdot 57\ \mathrm{k\Omega} \cdot 4{,}7\ \mathrm{\mu F} = 0{,}186\ \mathrm{s}$$

$$t_L = 0{,}693\,R_B\,C = 0{,}693 \cdot 47\ \mathrm{k\Omega} \cdot 4{,}7\ \mathrm{\mu F} = 0{,}153\ \mathrm{s}$$

$$T = t_H + t_L = 0{,}339\ \mathrm{s} \;\Rightarrow\; f = \frac{1}{T} = 2{,}95\ \mathrm{Hz}$$

$$\text{činiteľ plnenia} = \frac{t_H}{T} = 55\ \%$$

**Výsledok:** $f_{U3} = 2{,}95\ \mathrm{Hz}$ (približne 3 pípnutia za sekundu).

---

## 10. Rýchly oscilátor U2 (tón)

**Čo počítame:** frekvenciu tónu. $R_A = 4{,}7\ \mathrm{k\Omega}$, $R_B = 22\ \mathrm{k\Omega}$, $C = 10\ \mathrm{nF}$.

$$t_H = 0{,}693 \cdot (4{,}7 + 22)\ \mathrm{k\Omega} \cdot 10\ \mathrm{nF} = 185\ \mathrm{\mu s}$$

$$t_L = 0{,}693 \cdot 22\ \mathrm{k\Omega} \cdot 10\ \mathrm{nF} = 152\ \mathrm{\mu s}$$

$$T = 337\ \mathrm{\mu s} \;\Rightarrow\; f = \frac{1}{T} = 2{,}96\ \mathrm{kHz}$$

**Výsledok:** $f_{U2} = 2{,}96\ \mathrm{kHz}$.

---

## 11. Najväčší prúd cez Rp

**Čo počítame:** maximálny prúd z výstupu U2 do piezo meniča (limituje ho Rp).

$$I_{max} = \frac{V_{CC}}{R_p} = \frac{8{,}7}{100} = 87\ \mathrm{mA}$$

**Výsledok:** 87 mA, čo je pod hranicou výstupu NE555 (200 mA).

---

## 12. Výdrž batérie (orientačne)

**Čo počítame:** ako dlho vydrží 9 V batéria. Odhad spotreby: 3 × NE555 (3 – 6 mA každý) + LED (≈ 15 mA) ≈ 30 mA. Kapacita alkalickej 9 V batérie ≈ 500 mAh.

$$t = \frac{500\ \mathrm{mAh}}{30\ \mathrm{mA}} \approx 17\ \mathrm{h}$$

**Výsledok:** približne 17 hodín (hrubý odhad).

---

## Súhrn výsledkov

| Veličina | Hodnota |
|---|---|
| Napájanie za D2 | 8,7 V |
| Zapnutie lampy ($R_{LDR}$) | 66 kΩ (≈ 0,3 – 1,4 lux) |
| Vypnutie lampy ($R_{LDR}$) | 16,5 kΩ (≈ 3 – 14 lux) |
| Prúd LED | 14,9 mA |
| Výkon na Rl | 86 mW |
| Frekvencia rytmu (U3) | 2,95 Hz |
| Frekvencia tónu (U2) | 2,96 kHz |
| Max. prúd cez Rp | 87 mA |
| Výdrž batérie (odhad) | ≈ 17 h |

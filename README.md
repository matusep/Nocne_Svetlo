# Nočné svetlo s ranným budíkom

Semestrálny projekt z predmetu **Základy elektroniky**.
Autor: *(meno, študijná skupina)*

Obvod sám rozsvieti LED lampu, keď sa zotmie, a ráno, keď sa rozjasní, začne pípať piezo meničom. Je postavený z troch časovačov NE555, dvoch tranzistorov a fotorezistora. Doska (50 × 40 mm, 2 vrstvy) je navrhnutá v EasyEDA Pro a osadzuje ju JLCPCB (Economic assembly). Fotorezistor LDR1 JLCPCB neosádza, spája sa ručne.

Napájanie: 9 V batéria cez konektor JST PH2.0.

---

## 1. Ako to funguje

```
svetlo -> LDR1 + Rf (delič) -> U1 (Schmittov spúšťač) -> OUT1
OUT1 = HIGH (tma)    -> Q1 zapne LED lampu (D1)
OUT1 = LOW  (svetlo) -> Q2 sa vypne, DEN = HIGH -> U3 (rytmus) -> U2 (tón) -> piezo BZ1
```

| Situácia | LDR1 | Bod X | OUT1 | Lampa D1 | DEN | Budík |
|---|---|---|---|---|---|---|
| **Tma** | veľký odpor | pod 1/3 Vcc | HIGH | svieti | ≈ 0 V | ticho (U3 v resete) |
| **Svetlo** | malý odpor | nad 2/3 Vcc | LOW | nesvieti | ≈ Vcc | pípa |

> Budík nemá hodiny, reaguje len na svetlo. Pípa preto vždy, keď je svetlo (aj cez deň).

---

## 2. Súčiastky a ich funkcia

### Napájanie

| Označenie | Hodnota | LCSC | Funkcia |
|---|---|---|---|
| J1 | JST PH2.0 2-pin | C54582899 | Vstup z batérie (pin 1 = +, pin 2 = GND) |
| D2 | SS14 | C55127344 | Ochrana proti opačnej polarite |
| C4, C5, C6 | 100 nF | C1591 | Blokovanie napájania pri každom NE555 |
| C7 | 10 µF | C15850 | Zásoba energie pre celú dosku |

### Senzor a rozhodovanie

| Označenie | Hodnota | LCSC | Funkcia |
|---|---|---|---|
| LDR1 | GT36528 | C2904881 | Fotorezistor (senzor svetla) |
| Rf | 33 kΩ | C4216 | Spolu s LDR1 tvorí delič napätia |
| U1 | NE555 | C5125085 | Schmittov spúšťač (rozhoduje tma / svetlo) |
| C1, C2c, C3c | 10 nF | C1589 | Filter rušenia na pine 5 (U1, U3, U2) |

### Lampa

| Označenie | Hodnota | LCSC | Funkcia |
|---|---|---|---|
| Rb1 | 10 kΩ | C25804 | Prúd do bázy Q1 |
| Q1 | 2N3904 | C53311656 | Spínač LED |
| Rl | 390 Ω | C23151 | Obmedzenie prúdu LED |
| D1 | YLED0603W | C19273150 | LED lampa |

### Budík

| Označenie | Hodnota | LCSC | Funkcia |
|---|---|---|---|
| Rb2 | 47 kΩ | C25819 | Prúd do bázy Q2 |
| Q2 | 2N3904 | C53311656 | Invertor (z OUT1 robí DEN) |
| Rc2 | 10 kΩ | C25804 | Pull-up kolektora Q2 |
| U3 | NE555 | C5125085 | Pomalý oscilátor (rytmus pípania) |
| RT3A, RT3B | 10 kΩ, 47 kΩ | C25804, C25819 | Časovanie U3 |
| CT3 | 4,7 µF | C1779 | Časovanie U3 |
| U2 | NE555 | C5125085 | Rýchly oscilátor (tón) |
| RT2A, RT2B | 4,7 kΩ, 22 kΩ | C23162, C31850 | Časovanie U2 |
| CT2 | 10 nF | C1589 | Časovanie U2 |
| Rp | 100 Ω | C22775 | Ochrana výstupu U2 pred prúdovými špičkami |
| BZ1 | KLJ-1230 (pasívny piezo) | C201042 | Zvuk |

---

## 3. Spojenia (netlist)

| Uzol | Čo je na ňom pripojené |
|---|---|
| **+9V** (po D2) | U1 p4, p8; U2 p8; U3 p8; LDR1; Rl; Rc2; RT3A; RT2A; C4–C7 |
| **GND** | U1, U2, U3 p1; Rf; C1, C2c, C3c; CT2, CT3; emitory Q1 a Q2; BZ1; J1 p2; C4–C7 |
| **X** | U1 p2 a p6; LDR1; Rf |
| **OUT1** | U1 p3; Rb1; Rb2 |
| **DEN** | kolektor Q2; Rc2; U3 p4 |
| **OUT3** | U3 p3; U2 p4 |

Ďalej: D2 anóda → J1 p1 (VBAT), katóda → +9V. D1 anóda cez Rl na +9V, katóda na kolektor Q1. U1 p7 a J1 p3, p4 sú nezapojené.

---

## 4. Výpočty

### 4.1 Napájacie napätie

Na dióde D2 (SS14) klesne približne 0,3 V.

$$V_{CC} = V_{bat} - V_{D2} = 9 - 0{,}3 = 8{,}7\ \mathrm{V}$$

### 4.2 Delič napätia LDR1 + Rf

LDR1 je medzi $V_{CC}$ a bodom X, Rf je medzi X a GND.

$$X = V_{CC} \cdot \frac{R_f}{R_f + R_{LDR}}$$

Na svetle je $R_{LDR}$ malý, takže X je vysoko. V tme je $R_{LDR}$ veľký, takže X je nízko.

### 4.3 Prahy Schmittovho spúšťača U1

NE555 prepína pri $\tfrac{1}{3}V_{CC}$ a $\tfrac{2}{3}V_{CC}$ (vstupy 2 a 6 sú spojené do bodu X).

**Horný prah** (svetlo, lampa sa vypne):

$$\frac{R_f}{R_f + R_{LDR}} = \frac{2}{3} \;\;\Rightarrow\;\; R_{LDR} = \frac{R_f}{2} = \frac{33}{2} = 16{,}5\ \mathrm{k\Omega}$$

**Dolný prah** (tma, lampa sa zapne):

$$\frac{R_f}{R_f + R_{LDR}} = \frac{1}{3} \;\;\Rightarrow\;\; R_{LDR} = 2R_f = 2 \cdot 33 = 66\ \mathrm{k\Omega}$$

Prahy nezávisia od napätia batérie, lebo X aj prahy sú úmerné $V_{CC}$.

**Kontrola** pri $V_{CC} = 8{,}7\ \mathrm{V}$:

$$R_{LDR} = 16{,}5\ \mathrm{k\Omega}:\quad X = 8{,}7 \cdot \frac{33}{33 + 16{,}5} = 5{,}8\ \mathrm{V} = \tfrac{2}{3}\cdot 8{,}7\ \mathrm{V}$$

$$R_{LDR} = 66\ \mathrm{k\Omega}:\quad X = 8{,}7 \cdot \frac{33}{33 + 66} = 2{,}9\ \mathrm{V} = \tfrac{1}{3}\cdot 8{,}7\ \mathrm{V}$$

Medzera medzi prahmi (hysterézia) zabraňuje blikaniu lampy v súmraku.

### 4.4 Pri akom osvetlení sa obvod prepne

Pre GT36528 katalóg udáva $R_{10} = 8$ až $20\ \mathrm{k\Omega}$ pri 10 lux a $\gamma = 0{,}6$.

$$R = R_{10}\left(\frac{10}{E}\right)^{\gamma} \;\;\Rightarrow\;\; E = 10\left(\frac{R_{10}}{R}\right)^{1/\gamma}$$

Príklad (zapnutie lampy, $R = 66\ \mathrm{k\Omega}$, $R_{10} = 8\ \mathrm{k\Omega}$):

$$E = 10 \cdot \left(\frac{8}{66}\right)^{1/0{,}6} = 0{,}30\ \mathrm{lux}$$

| Prepnutie | $R_{LDR}$ | $E$ pri $R_{10} = 8\ \mathrm{k\Omega}$ | $E$ pri $R_{10} = 20\ \mathrm{k\Omega}$ |
|---|---|---|---|
| Lampa sa **zapne** (tma) | 66 kΩ | 0,30 lux | 1,4 lux |
| Lampa sa **vypne** (svetlo) | 16,5 kΩ | 3,0 lux | 13,8 lux |

Presná hodnota závisí od konkrétneho kusu fotorezistora (výrobný rozptyl).

### 4.5 Lampa (Q1, D1, Rl)

Predpoklad: napätie na LED $V_f \approx 2{,}8\ \mathrm{V}$ (odhad, nemerané) a $V_{CE(sat)} \approx 0{,}1\ \mathrm{V}$.

**Prúd LED:**

$$I_{LED} = \frac{V_{CC} - V_f - V_{CE(sat)}}{R_l} = \frac{8{,}7 - 2{,}8 - 0{,}1}{390} = 14{,}9\ \mathrm{mA}$$

**Báza Q1.** Výstup NE555 v stave HIGH je približne $V_{CC} - 1{,}7 = 7{,}0\ \mathrm{V}$.

$$I_{B1} = \frac{7{,}0 - 0{,}7}{10\ \mathrm{k\Omega}} = 0{,}63\ \mathrm{mA}$$

$$\beta_{\text{nútené}} = \frac{I_{LED}}{I_{B1}} = \frac{14{,}9}{0{,}63} \approx 24$$

Nútené $\beta$ (24) je menšie než $h_{FE}$ tranzistora 2N3904 (typicky $\geq 100$), takže je Q1 v saturácii a funguje ako spínač.

**Výkon na Rl:**

$$P_{R_l} = I_{LED}^{2} \cdot R_l = (0{,}0149)^2 \cdot 390 = 86\ \mathrm{mW}$$

Odpor 0603 má menovitý výkon 100 mW, takže je v limite, ale s malou rezervou. Môže sa zohriať.

### 4.6 Invertor (Q2, Rb2, Rc2)

$$I_{B2} = \frac{7{,}0 - 0{,}7}{47\ \mathrm{k\Omega}} = 0{,}134\ \mathrm{mA}$$

$$I_{C2} = \frac{V_{CC} - V_{CE(sat)}}{R_{c2}} = \frac{8{,}7 - 0{,}1}{10\ \mathrm{k\Omega}} = 0{,}86\ \mathrm{mA}$$

$$\beta_{\text{nútené}} = \frac{0{,}86}{0{,}134} \approx 6{,}4 \;\ll\; h_{FE}$$

Q2 je teda v saturácii.

- **Tma** (OUT1 = HIGH): Q2 vedie, DEN ≈ 0,1 V, čo je pod hranicou resetu NE555 (pod 0,4 V). U3 je vypnutý.
- **Svetlo** (OUT1 = LOW): Q2 nevedie, Rc2 vytiahne DEN na $V_{CC}$. U3 beží.

### 4.7 Pomalý oscilátor U3 (rytmus)

Astabilný režim NE555: $R_A = 10\ \mathrm{k\Omega}$, $R_B = 47\ \mathrm{k\Omega}$, $C = 4{,}7\ \mathrm{\mu F}$.

$$t_H = 0{,}693\,(R_A + R_B)\,C = 0{,}693 \cdot 57\ \mathrm{k\Omega} \cdot 4{,}7\ \mathrm{\mu F} = 0{,}186\ \mathrm{s}$$

$$t_L = 0{,}693\,R_B\,C = 0{,}693 \cdot 47\ \mathrm{k\Omega} \cdot 4{,}7\ \mathrm{\mu F} = 0{,}153\ \mathrm{s}$$

$$T = t_H + t_L = 0{,}339\ \mathrm{s} \;\;\Rightarrow\;\; f = \frac{1}{T} = 2{,}95\ \mathrm{Hz}$$

$$\text{činiteľ plnenia} = \frac{t_H}{T} = 55\ \%$$

Približne 3 pípnutia za sekundu.

### 4.8 Rýchly oscilátor U2 (tón)

$R_A = 4{,}7\ \mathrm{k\Omega}$, $R_B = 22\ \mathrm{k\Omega}$, $C = 10\ \mathrm{nF}$.

$$t_H = 0{,}693 \cdot (4{,}7 + 22)\ \mathrm{k\Omega} \cdot 10\ \mathrm{nF} = 185\ \mathrm{\mu s}$$

$$t_L = 0{,}693 \cdot 22\ \mathrm{k\Omega} \cdot 10\ \mathrm{nF} = 152\ \mathrm{\mu s}$$

$$T = 337\ \mathrm{\mu s} \;\;\Rightarrow\;\; f = \frac{1}{T} = 2{,}96\ \mathrm{kHz}$$

Pin 4 (reset) U2 je pripojený na OUT3, takže tón znie len vtedy, keď je OUT3 = HIGH. Vzniká „bip-bip-bip".

### 4.9 Ochrana výstupu (Rp)

Pri nabíjaní kapacity piezo meniča prúd z výstupu U2 nepresiahne:

$$I_{max} = \frac{V_{CC}}{R_p} = \frac{8{,}7}{100} = 87\ \mathrm{mA}$$

NE555 vydrží na výstupe do 200 mA, takže je výstup v bezpečí.

### 4.10 Spotreba a výdrž batérie (orientačne)

Každý NE555 si pri 9 V berie niekoľko mA (rádovo 3 – 6 mA), plus LED približne 15 mA. V tme je spotreba rádovo 30 mA.

$$t_{\text{výdrž}} \approx \frac{500\ \mathrm{mAh}}{30\ \mathrm{mA}} \approx 17\ \mathrm{h}$$

Je to hrubý odhad pre alkalickú 9 V batériu. Obvod nemá vypínač, vypína sa vytiahnutím konektora z J1.

### 4.11 Súhrn výsledkov

| Veličina | Hodnota |
|---|---|
| Napájanie za D2 | 8,7 V |
| Prah zapnutia lampy ($R_{LDR}$) | 66 kΩ (≈ 0,3 – 1,4 lux) |
| Prah vypnutia lampy ($R_{LDR}$) | 16,5 kΩ (≈ 3 – 14 lux) |
| Prúd LED | 14,9 mA |
| Výkon na Rl | 86 mW |
| Frekvencia rytmu (U3) | 2,95 Hz |
| Frekvencia tónu (U2) | 2,96 kHz |
| Max. prúd z U2 do piezo | 87 mA |
| Výdrž batérie (odhad) | ≈ 17 h |

---

## 5. Známe obmedzenia

- Budík pípa vždy, keď je svetlo (nemá hodiny ani časovač).
- LED môže svietiť na fotorezistor a ovplyvniť ho. Hysterézia to väčšinou ošetrí, pomôže aj zatienenie LDR (napr. čierna páska).
- Hodnoty $V_f$ LED a max. prúdu LED sú odhady, nie merania z datasheetu.
- Odpor Rl 390 Ω (0603) je zaťažený približne 86 mW zo 100 mW.
- Piezo má maximum hlasitosti pri rezonancii (okolo 4 kHz), tón 2,96 kHz je trochu pod ňou, takže je menej hlasný.
- Správna polarita kábla k J1: červený (+) vodič na pin 1.
- Skutočné správanie sa môže od výpočtov líšiť (tolerancie súčiastok, klon NE555).

---

## 6. Súbory

- `schema_zapojenia.png` – schéma zapojenia
- `simulacia_priebehy.png` – priebehy zo simulácie
- `nocne_svetlo_circuitjs.txt` – obvod pre simulátor CircuitJS (File → Import)
- gerber / BOM / CPL – výrobné súbory z EasyEDA

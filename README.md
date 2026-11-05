# Nočné svetlo s ranným budíkom

Semestrálny projekt z predmetu **Základy elektroniky**.
Autor: *(meno, študijná skupina)*

Obvod sám rozsvieti LED lampu, keď sa zotmie, a ráno, keď sa rozjasní, začne pípať piezo meničom. Je postavený z troch časovačov NE555, dvoch tranzistorov a fotorezistora. Doska (50 × 40 mm, 2 vrstvy) je navrhnutá v EasyEDA Pro a osadzuje ju JLCPCB (Economic assembly). Fotorezistor LDR1 JLCPCB neosádza, spája sa ručne.

Napájanie: 9 V batéria cez konektor JST PH2.0.

---

## 1. Ako to funguje (stručne)

```
svetlo -> LDR1 + Rf (delič) -> U1 (Schmittov spúšťač) -> OUT1
OUT1 = HIGH (tma)   -> Q1 zapne LED lampu (D1)
OUT1 = LOW  (svetlo)-> Q2 sa vypne, DEN = HIGH -> U3 (pomalý rytmus) -> U2 (tón) -> piezo BZ1
```

| Situácia | LDR1 | Bod X | OUT1 | Lampa D1 | DEN | Budík | 
|---|---|---|---|---|---|---|
| Tma | veľký odpor | < 1/3 Vcc | HIGH | svieti | ≈ 0 V | ticho (U3 v resete) |
| Svetlo | malý odpor | > 2/3 Vcc | LOW | nesvieti | ≈ Vcc | pípa |

Poznámka: budík nemá hodiny, reaguje len na svetlo. Pípa preto vždy, keď je svetlo (aj cez deň).

---

## 2. Zoznam súčiastok a čo robia

| Označenie | Hodnota / typ | LCSC | Funkcia |
|---|---|---|---|
| J1 | JST PH2.0 2-pin (XY-S2B-PH-SM4-TB) | C54582899 | Vstup z 9 V batérie (pin 1 = +, pin 2 = GND) |
| D2 | SS14 (Schottkyho dióda) | C55127344 | Ochrana proti opačnej polarite batérie |
| C4, C5, C6 | 100 nF | C1591 | Blokovanie napájania pri každom NE555 |
| C7 | 10 µF | C15850 | Zásoba energie pre celú dosku |
| LDR1 | GT36528 (fotorezistor) | C2904881 | Senzor svetla |
| Rf | 33 kΩ | C4216 | Spolu s LDR1 tvorí delič napätia |
| U1 | NE555 (SOIC-8) | C5125085 | Schmittov spúšťač (rozhoduje tma / svetlo) |
| C1, C2c, C3c | 10 nF | C1589 | Filter rušenia na pine 5 (CONTROL) U1, U3, U2 |
| Rb1 | 10 kΩ | C25804 | Prúd do bázy Q1 |
| Q1 | 2N3904 | C53311656 | Spínač LED |
| Rl | 390 Ω | C23151 | Obmedzenie prúdu LED |
| D1 | LED YLED0603W | C19273150 | Lampa |
| Rb2 | 47 kΩ | C25819 | Prúd do bázy Q2 |
| Q2 | 2N3904 | C53311656 | Invertor (z OUT1 robí DEN) |
| Rc2 | 10 kΩ | C25804 | Pull-up kolektora Q2 |
| U3 | NE555 | C5125085 | Pomalý oscilátor (rytmus pípania) |
| RT3A / RT3B | 10 kΩ / 47 kΩ | C25804 / C25819 | Časovanie U3 |
| CT3 | 4,7 µF | C1779 | Časovanie U3 |
| U2 | NE555 | C5125085 | Rýchly oscilátor (tón) |
| RT2A / RT2B | 4,7 kΩ / 22 kΩ | C23162 / C31850 | Časovanie U2 |
| CT2 | 10 nF | C1589 | Časovanie U2 |
| Rp | 100 Ω | C22775 | Ochrana výstupu U2 pred prúdovými špičkami |
| BZ1 | piezo KLJ-1230 (pasívny) | C201042 | Zvuk |

---

## 3. Zapojenie (spojenia)

- **+9V (po D2):** U1 p4, p8; U2 p8; U3 p8; LDR1; Rl; Rc2; RT3A; RT2A; C4–C7
- **GND:** U1, U2, U3 p1; Rf; C1, C2c, C3c; CT2, CT3; emitory Q1 a Q2; BZ1; J1 p2; C4–C7
- **X:** U1 p2 a p6; LDR1; Rf
- **OUT1:** U1 p3; Rb1; Rb2
- **DEN:** kolektor Q2; Rc2; U3 p4
- **OUT3:** U3 p3; U2 p4
- Ďalej: D2 anóda -> J1 p1 (VBAT), katóda -> +9V; D1 anóda cez Rl na +9V, katóda na kolektor Q1; U1 p7 nezapojený; J1 p3, p4 nezapojené.

---

## 4. Výpočty

### 4.1 Napájanie

Batéria 9 V, na D2 (SS14) klesne približne 0,3 V:

```
Vcc = 9 V - 0,3 V = 8,7 V
```

### 4.2 Delič LDR1 + Rf (bod X)

LDR1 je medzi +Vcc a bodom X, Rf je medzi X a GND:

```
X = Vcc * Rf / (Rf + R_LDR)
```

### 4.3 Prahy Schmittovho spúšťača U1

NE555 prepína pri 1/3 a 2/3 napájania (vstupy 2 a 6 sú spojené):

```
Horný prah:  X = 2/3 Vcc  ->  Rf/(Rf+R) = 2/3  ->  R_LDR = Rf/2 = 16,5 kΩ  (svetlo, lampa sa vypne)
Dolný prah:  X = 1/3 Vcc  ->  Rf/(Rf+R) = 1/3  ->  R_LDR = 2*Rf = 66 kΩ    (tma, lampa sa zapne)
```

Prahy nezávisia od napätia batérie, lebo X aj prahy sú úmerné Vcc. Kontrola pri Vcc = 8,7 V:

```
R_LDR = 16,5 kΩ: X = 8,7 * 33/(33+16,5) = 5,8 V  (= 2/3 * 8,7 V)
R_LDR = 66 kΩ:   X = 8,7 * 33/(33+66)   = 2,9 V  (= 1/3 * 8,7 V)
```

Rozdiel medzi prahmi (hysterézia) zabraňuje blikaniu lampy v súmraku.

### 4.4 Pri akom osvetlení sa to prepne

Pre GT36528 katalóg udáva 8 – 20 kΩ pri 10 lux a γ = 0,6. Pre fotorezistor platí:

```
R = R10 * (10 / E)^γ     ->     E = 10 * (R10 / R)^(1/γ)
```

| | R_LDR | E pri R10 = 8 kΩ | E pri R10 = 20 kΩ |
|---|---|---|---|
| Lampa sa zapne (tma) | 66 kΩ | 0,30 lux | 1,4 lux |
| Lampa sa vypne (svetlo) | 16,5 kΩ | 3,0 lux | 13,8 lux |

Presné hodnoty závisia od kusu fotorezistora (výrobný rozptyl). Lampa sa teda zapne pri veľmi slabom osvetlení a vypne pri svetle rádovo 3 – 14 lux (svit oblohy na úsvite).

### 4.5 Lampa (Q1, D1, Rl)

Predpoklad: napätie na LED Vf ≈ 2,8 V (odhad, nemerané), Vce(sat) tranzistora ≈ 0,1 V.

```
I_LED = (Vcc - Vf - Vce_sat) / Rl = (8,7 - 2,8 - 0,1) / 390 = 14,9 mA
```

Výstup NE555 v stave HIGH je približne Vcc − 1,7 V = 7,0 V:

```
I_B1 = (7,0 - 0,7) / 10 kΩ = 0,63 mA
Nútené β = I_LED / I_B1 = 14,9 / 0,63 ≈ 24
```

Nútené β (24) je menšie než hFE tranzistora 2N3904 (typicky ≥ 100), takže je Q1 v saturácii a funguje ako spínač.

Výkon na Rl:

```
P_Rl = I^2 * R = 0,0149^2 * 390 = 86 mW
```

Odpor 0603 má menovitý výkon 100 mW, takže je to v limite, ale s malou rezervou. Odpor sa môže zohriať.

### 4.6 Invertor (Q2, Rb2, Rc2)

```
I_B2 = (7,0 - 0,7) / 47 kΩ = 0,134 mA
I_C2 = (Vcc - Vce_sat) / Rc2 = (8,7 - 0,1) / 10 kΩ = 0,86 mA
Nútené β = 0,86 / 0,134 ≈ 6,4  (<< hFE, tranzistor je v saturácii)
```

- Tma (OUT1 = HIGH): Q2 vedie, DEN ≈ 0,1 V, čo je pod hranicou resetu NE555 (< 0,4 V). U3 je vypnutý.
- Svetlo (OUT1 = LOW): Q2 nevedie, Rc2 vytiahne DEN na Vcc. U3 beží.

### 4.7 Pomalý oscilátor U3 (rytmus)

Astabilný režim NE555: RT3A = 10 kΩ, RT3B = 47 kΩ, CT3 = 4,7 µF.

```
t_H = 0,693 * (RA + RB) * C = 0,693 * 57 kΩ * 4,7 µF = 0,186 s
t_L = 0,693 * RB * C        = 0,693 * 47 kΩ * 4,7 µF = 0,153 s
T = t_H + t_L = 0,339 s
f = 1/T = 2,95 Hz      (vzorec f = 1,44 / ((RA + 2RB) C) dáva 2,95 Hz)
Činiteľ plnenia = t_H / T = 55 %
```

Približne 3 pípnutia za sekundu.

### 4.8 Rýchly oscilátor U2 (tón)

RT2A = 4,7 kΩ, RT2B = 22 kΩ, CT2 = 10 nF.

```
t_H = 0,693 * (4,7 kΩ + 22 kΩ) * 10 nF = 185 µs
t_L = 0,693 * 22 kΩ * 10 nF            = 152 µs
T = 337 µs
f = 1/T = 2,96 kHz     (vzorec: 1,44 / ((RA + 2RB) C) = 2,96 kHz)
Činiteľ plnenia ≈ 55 %
```

U2 má pin 4 (reset) pripojený na OUT3, takže tón znie len vtedy, keď je OUT3 = HIGH, a vzniká „bip-bip-bip".

### 4.9 Ochrana výstupu (Rp)

Pri nabíjaní kapacity piezo meniča prúd z výstupu U2 nepresiahne:

```
I_max = Vcc / Rp = 8,7 / 100 = 87 mA
```

NE555 vydrží na výstupe do 200 mA, takže je výstup v bezpečí.

### 4.10 Spotreba a výdrž batérie (orientačne)

Každý NE555 si pri 9 V berie niekoľko mA (rádovo 3 – 6 mA, odhad z katalógu), plus LED ≈ 15 mA. V tme je spotreba rádovo 30 mA:

```
Výdrž ≈ 500 mAh / 30 mA ≈ 17 hodín
```

Je to hrubý odhad pre alkalickú 9 V batériu. Obvod nemá vypínač, preto sa vypína vytiahnutím konektora z J1.

---

## 5. Známe obmedzenia

- Budík pípa vždy, keď je svetlo (nemá hodiny ani časovač).
- LED môže svietiť na fotorezistor a ovplyvniť ho. Hysterézia to väčšinou ošetrí, pomôže aj zatienenie LDR (napr. čierna páska).
- Hodnoty Vf LED a max. prúdu LED sú odhady, nie merania z datasheetu.
- Odpor Rl 390 Ω (0603) je zaťažený približne 86 mW z 100 mW.
- Piezo má maximum hlasitosti pri rezonancii (okolo 4 kHz), tón 2,96 kHz je trochu pod ním, čo je menej hlasné.
- Správna polarita kábla k J1: červený (+) vodič na pin 1.
- Skutočné správanie sa môže od výpočtov líšiť (tolerancie súčiastok, klon NE555).

---

## 6. Súbory

- `schema_zapojenia.png` – schéma zapojenia
- `simulacia_priebehy.png` – priebehy zo simulácie
- `nocne_svetlo_circuitjs.txt` – obvod pre simulátor CircuitJS (File → Import)
- gerber / BOM / CPL – výrobné súbory z EasyEDA

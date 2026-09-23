# Handoff: Gantt Chart Planner – iz v2.0 v v3.0

**Ustvarjeno:** 2026-09-23 22:14 (Europe/Ljubljana)
**Branch:** ni git repozitorija (ena sama datoteka HTML)
**Trajanje seje:** en daljši pogovor (4 krogi intervjuja + gradnja v1.0 in v2.0)
**Stanje konteksta:** poraba je visoka (zelo dolg pogovor z veliko klici orodij), natančne številke niso zanesljivo zaznavne. Predajo je ročno zahteval uporabnik, preden se začne v3.0. Nobena akcija ni ostala nedokončana.

---

## Povzetek

Samostojna spletna aplikacija za Gantt diagram (podmnožica MS Projecta) v **eni datoteki HTML**: brez namestitve, brez administratorskih pravic, deluje brez povezave, prilagojena telefonu. **Različica 2.0 je predana in testirana** (Chromium). Naslednji korak je **v3.0** s petimi novimi funkcijami. Načrt je predlagan, uporabnik pa še ni odgovoril na 3 odprta vprašanja (glej *Odprta vprašanja*).

**Pomembno za novo okno:** priloži datoteko `gantt-planner.html` (v2.0) v novi pogovor. Delovni prostor seje se ne prenese.

---

## Pravila uporabnika (iz njegovih nastavitev, OBVEZNO)

Odgovarjaj v slovenščini, v nevtralnem tonu. Pred vsako izvedbo predstavi kratek načrt in počakaj na potrditev. Ob vsakem koraku napiši zapisnik v obliki za copy-paste. Na začetek odgovora daj eno vrstico z oceno stroška, modelom in effortom. Opozori na tveganje stiskanja konteksta. Datoteke in predloge verzioniraj (1.0 → 1.1 manjše, → 2.0 večje spremembe). Dodaj glosar novih izrazov (EN/DE/FR/IT) in oceno tveganj v obliki tveganje / verjetnost / učinek / ublažitev. Na koncu daj 3 odebeljena nadaljevalna vprašanja. Kadar nečesa ne veš, napiši »ne vem«. Predpostavke označi s PREDPOSTAVKA. **Vmesnik aplikacije je v angleščini.**

---

## Dnevnik razvojnega procesa

1. Zajem zahtev: grill ×3 (8 + 25 + 30 vprašanj) + grillme (34 vprašanj), ~101 odločitev. Stanje: **opravljeno**. Nastalo: odločitve v pogovoru (povzete spodaj).
2. Gradnja v1.0 (jedro, urnikovanje, tabela, časovnica, uvoz in izvoz, tisk). Stanje: **opravljeno**. Nastalo: `gantt-planner.html` v1.0.
3. QA v1.0: 3 iteracije v Playwright/Chromium, 6 napak popravljenih. Stanje: **opravljeno**.
4. Uporabnikov test v1.0 v Safariju na macOS 26.6.2. Stanje: **opravljeno**, brez prijavljenih napak.
5. Gradnja v2.0 (primeri povezav, vlečenje povezav z miško, prazniki SI/US, rahel kontrast, ISO tedni v mesečnem pogledu, omejitev SNET). Stanje: **opravljeno**. Nastalo: `gantt-planner.html` v2.0 (~142 KB, 2779 vrstic).
6. QA v2.0 (Chromium). Stanje: **opravljeno**. Safari za v2.0 še ni preverjen.
7. Načrt v3.0. Stanje: **v teku**, predlog je dan, čaka na odgovore.
8. Gradnja v3.0. Stanje: **ni začeto**.
9. QA v3.0 in predaja. Stanje: **ni začeto**.

Skupni napredek: 6 od 9 faz.

---

## Ključne odločitve (izbor, veljavne v v2.0)

| Odločitev | Razlog |
|---|---|
| En `.html`, vse vgrajeno, brez CDN in brez localStorage | prenosljivost, zaklenjena korporativna okolja |
| Shranjevanje = izvoz/uvoz JSON (format **1.1**, bere 1.x) | brez namestitve; opozorilo `beforeunload`, ko so spremembe neshranjene |
| Faze (2 ravni) + naloge + mejniki (krog, brez povezav, kljukica »reached«) | izbira uporabnika |
| Vsi tipi povezav FS/SS/FF/SF + lag/lead, samodejno razporejanje (ASAP), preprečevanje ciklov | kot MS Project |
| Koledarski dnevi, najmanj 1 dan, cela števila; prazniki so SAMO prikazani | izbira uporabnika (odločitev o tem, da bi štele delovne dni, je še odprta) |
| **SNET** (»Start no earlier than«) na povezani nalogi: vpis začetka v tabeli nastavi omejitev, prazno polje jo odstrani, v celici je oranžna oznaka | dobra praksa (MS Project, DCMA) |
| Vlečenje povezav samo z miško ali pisalom; na dotik ostane urejanje v oknu | brez konflikta z drsenjem |
| Undo 10 korakov (tudi uvoz), ne pokaže modalnega okna | izbira uporabnika |
| Obvestila so modalna okna z OK | izbira uporabnika |
| Izvoz: JSON, CSV (»;« + BOM, ID kot `="1.2"`), osnovni MS Project XML (MSPDI, 7-dnevni koledar, SNET) | Excel s slovenskimi nastavitvami; uporabnik bo XML preveril v MS Projectu |
| Tisk: A4 ležeče, razdelitev na več strani (vrstice × čas), glava in noga | izbira uporabnika |
| Ctrl+N ni mogoč (rezervira ga brskalnik) → **Alt+N** | omejitev brskalnika |

---

## Datoteke

### Ustvarjene
- `gantt-planner.html` (v2.0): celotna aplikacija. Bloki `<script>` (vrstice v2.0):
  - `core-utils` (366): datumi kot »številke dni« v UTC. `isoToDay`, `dayToIso`, `dayToDisp` (DD.MM.YYYY), `dispToDay` (strpen razčlenjevalnik vnosa), `mondayIndex` (pon=0), `firstOfMonth/Year`, `esc`, `clampInt`, `safeFileName`, `download` (prenos prek Blob).
  - `core-holidays` (490): `easterSunday` (gregorijanski algoritem), `nthWeekday`, `lastWeekday`, `holidaysForYear(cc,y)` (SI: 15 dela prostih dni; US: 11 zveznih, dan praznovanja premaknjen z vikenda), `holidayIndex(from,to)` (Map dan → imena, upošteva `state.holidays.off`), `defaultHolidays()` = `{countries:['SI'],off:[]}`.
  - `core-model` (609): `state`, `commit(mutator)` (posnetek → sprememba → `schedule()` → korak za undo, če se je kaj spremenilo → izris), `undo/redo`, `numbering()` (1, 1.1 …), `phaseSummary`, `predText` (zapis kot v MS Projectu), `sampleProject()`.
  - `core-scheduler` (847): `depConstraint` (model z ekskluzivnim koncem), `topoOrder(tasks, depsOf)` (DFS, vrne cikel), `schedule()` (ASAP + SNET; naloga brez povezav obdrži svoj začetek), `dependentsOf(id)`.
  - `io` (968): `validateProject` (napake s točno potjo, npr. `phases[1].tasks[3].start`), `findJsonErrorPos` (lasten pregledovalnik za vrstico in stolpec), `importFile`, `exportJSON`, `exportCSV`, `exportXML`.
  - `ui-dialogs` (1306): `openDialog`/`showMessage`/`confirmBox` (nativni `<dialog>`), `openTaskDialog` (osnutek; »Save« je en korak za undo), `openPhaseDialog`, `openLinkDialog`, `openHolidaysDialog` (država + leto + kljukice), `depExamplesHTML` (primeri iz farmacevtskih strojnih inštalacij), `helpHTML`, `openWelcome`.
  - `ui-render` (1881): `timelineGeom` (week 22 px/dan, month 5 px/dan, `hdrH` 44/66, `hol`), `visibleRows` (skrčitev + iskanje), `timelineHeaderInner`, `backgroundInner` (zebra + prazniki), `gridPaths`, `arrowPath`, `timelineBodyInner(rows,g,rh,markerId,num,interactive)` (vrstni red plasti: ozadje → puščice → črte → konice puščic `.ahd` → ročice `.hdl`), `rowHTML`, `render()` (obdrži scroll in fokus prek `data-k`).
  - `ui-print` (2243): `buildPrint` (strani 1046×716 px), dogodka `beforeprint`/`afterprint`.
  - `ui-events` (2326): dejanja v orodni vrstici, `moveRow`, `add/delete/duplicate`, vlečenje povezav (`startLinkDrag`/`moveLinkDrag`/`endLinkDrag`/`applyLink`, `barUnderPointer`), bližnjice, `init()`.
- QA skripte (samo v tej seji, se ne prenesejo): `qa/t1.py`, `t3.py`, `t4.py` (Playwright Python).

---

## Pasti

- Orodje **Write** literalno pretvori `﻿` v nevidni znak BOM. Za takšne ubežne znake uporabi Python in `\\uFEFF`.
- Ločnice vrstic so na celicah (ne na `.row`), sicer puščice SVG proseva skozi pripete stolpce.
- Konice puščic `.ahd` so 10 px pred robom črte, da ne prekrijejo ročic `.hdl`.
- Na zaslonih na dotik: vrstica 45 px, pisava 16 px (iOS sicer poveča vnosna polja), gumbi ≥ 44 px. `render` se ob `resize` sproži le, če se spremeni postavitev stolpcev (tipkovnica na telefonu).
- Ali Safari/iOS deluje: **ne vem**. Datoteka HTML, odprta iz aplikacije Datoteke na iPhonu, je morda samo predogled brez JavaScripta.
- Novela ZPDPD (17. 2. 2026): kaj točno je spremenila, **ne vem**. Seznam dela prostih dni za 2026 se ujema z objavljenimi koledarji.
- Dva stara testa v `t1.py` ne uspeta namenoma (»dependent start is readonly«, »json version 1.0«): vedenje sem v v2.0 spremenil.

---

## Naslednji koraki

### Takoj (začni tukaj)
1. Uporabnik v novo okno priloži `gantt-planner.html` (v2.0) in ta dokument.
2. Dobi odgovore na **3 odprta vprašanja** (spodaj) in potrditev načrta v3.0.
3. Zgradi v3.0 (datotečni format → **1.2**, stari 1.x se še odpirajo):
   - **0. Črta za danes:** izrazita rdeča navpična črta + oznaka »Today« v glavi. Ob kliku na Today utripne (CSS animacija).
   - **1. Gumb »What do I do next«** desno od `#projName`. Za izbrano nalogo ali fazo (`ui.selected`) odpre okno s tremi polji v eni vrsti: **ena** predhodnica → izbrana → **ena** naslednica. Uporabnik izrecno želi SAMO prvo predhodnico in prvo naslednico, ne celotnih seznamov.
     - PREDPOSTAVKA (potrdi z uporabnikom): »prva predhodnica« = **odločilna predhodnica** (driving predecessor, kot v MS Project Task Path). To je povezava, katere omejitev (`depConstraint`) je dejansko določila začetek naloge; pri izenačenju prva na seznamu `deps`. Če je začetek določil SNET, to navedi.
     - »Prva naslednica« = naslednica z **najzgodnejšim začetkom**; pri izenačenju tista z nižjo številko (1.2 pred 1.3).
     - Pri vsakem polju: številka, ime, datumi, %, stanje (Done / In progress / Not started), tip povezave in lag.
     - Klik na predhodnico ali naslednico premakne pogled nanjo (hoja po verigi naprej ali nazaj). Če je ni, izpiši »No predecessor« oziroma »No successor«.
     - Pri fazi: prva predhodnica = odločilna zunanja predhodnica najzgodnejše naloge v fazi; prva naslednica = najzgodnejša zunanja naslednica. Če povezav ni, prejšnja ali naslednja faza po vrstnem redu.
     - Na časovnici poudari samo ti dve nalogi in njuni puščici.
   - **2. Spletne povezave** na nalogi ali fazi: `links:[{label,url}]`, dovoljeni samo `https:`, `http:` in `mailto:`. V tabeli ikona 🔗, klik odpre povezavo z `target=_blank rel=noopener`.
   - **3. Priponke:** način po odgovoru uporabnika (priporočeno: vgrajene v JSON kot base64, ≤ 5 MB na datoteko, prenos s klikom).
   - **4. Kalkulator delovnih dni** v zgornji vrstici (gumb 🧮 »Workdays« odpre vrstico): začetek (privzeto danes) + N delovnih dni → konec, ali konec → N. Kljukici »skip weekends« in »skip holidays« (uporabi `holidayIndex`). Štetje po odgovoru uporabnika.
4. QA: posodobi `t1.py` (pričakovanja iz v2.0) in dodaj teste za v3.0 (zabeleži namizje in 320 px posnetke zaslona).

### Pozneje
- Po želji uporabnika: prazniki in vikendi naj vplivajo na trajanje (delovni dnevi kot v MS Projectu); dodatne države (CH, DE).

---

## Odprta vprašanja (za uporabnika)
- [ ] **Priponke:** vgrajene v JSON (priporočeno) / samo povezava na datoteko / oboje?
- [ ] **Štetje delovnih dni v kalkulatorju:** kot MS Project, kjer je začetni dan 1. dan (priporočeno), ali »prištej N dni«, kjer začetni dan ne šteje?
- [ ] **Črta za danes:** vedno vidna + utripne ob kliku na Today (priporočeno), ali samo po kliku?
- [ ] **»What do I do next«:** potrdi, da je prva predhodnica = odločilna (tista, ki določa začetek), prva naslednica = tista z najzgodnejšim začetkom.
- [ ] Rezultat testa v2.0 v Safariju (vlečenje povezav z miško).
- [ ] Rezultat uvoza MS Project XML v pravi MS Project.

---

## Zapiski seje
Uporabnik: TS Engineer II (Novartis), CQV/farmacevtski projektni inženiring; testira na macOS Safari. Primere v aplikaciji oblikuj za strojne inštalacije v farmacevtskem okolju (brez imena podjetja v aplikaciji). Pri novih funkcijah upoštevaj dobro prakso (MS Project, DCMA).

_Predaja je nastala na zahtevo uporabnika pri visoki porabi konteksta. V novem oknu začni s tem dokumentom in priloženo datoteko v2.0._

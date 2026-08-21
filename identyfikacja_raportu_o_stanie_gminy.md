# Identyfikacja „raportu o stanie gminy / miasta”

Wnioski z ręcznej weryfikacji dokumentów, które **są** raportami o stanie jednostki (art. 28aa u.s.g.), ale pipeline oznaczył je `not_ok`. Cele:

1. potwierdzić, że treść dotyczy **właściwej** gminy (kolizje nazw);
2. gdy URL wygląda na stronę — wyłuskać **stały** URL pliku (bez tokenu sesji);
3. **potwierdzić rocznik** — rok **którego dotyczy** raport (nie rok publikacji / folderu BIP);
4. opisać, jak lepiej rozpoznawać raport.

Źródło: pobrane pliki, 2026-08-21. Siatka: 264 gminy × 2018–2025.

Art. 28aa: wójt/burmistrz/prezydent **do 31 maja roku X+1** przedstawia radzie raport **za rok X**. Gminy wydają raport za rok X w roku **X+1** (tytuł „maj 2023” przy raporcie za 2022).

Przyjęte jako `ok_finalnie`: 11 + 4 z tur tożsamości, plus 3 po **przeniesieniu na właściwy rok** (Brok 2022, Pisz 2022, Kobylin-Borzymy 2018), plus **23** z tury `identity:*` (§1e: OCR 3 stron, `catdoc`, rocznik z treści). Zostają 2× `wrong_doc_type` (Zambrów miasto 2020, Pozezdrze 2020) — pliki są raportami, psuje je hint POŚ/SRPS.

---

## 1. Werdykt tożsamości — pierwsza tura (11 URL-i)

| Jednostka (oczekiwana) | TERYT | Rok | Kolizja nazw | Werdykt | Dlaczego to ta gmina |
|---|---|---|---|---|---|
| Mały Płock, gmina wiejska, pow. kolneński, podlaskie | 2006042 | 2022, 2024, 2025 | **Płock** (miasto, mazowieckie) — nie ma w siatce, ale token „Płock” jest pułapką | **TAK, gmina wiejska Mały Płock** | Tytuł „Raport o stanie Gminy Mały Płock za ROK”; „województwa podlaskiego, w powiecie kolneńskim, 140 km²”; miejscowości **Chludnie, Korzeniste**. Brak „Miasta Płock”, starosty, mazowieckiego. Host `bip-ugmalyplock.podlaskie.eu`. |
| Szudziałowo, gmina wiejska, pow. sokólski | 2011102 | 2018 | brak w siatce | **TAK, Szudziałowo** | „Gmina Szudziałowo … województwa podlaskiego … granica z Białorusią”; granice Sokółka, Supraśl, Krynki, Gródek; 34 sołectwa / 43 miejscowości / ~3000 mieszkańców / 30 164 ha; **siedziba w miejscowości Szudziałowo**; **Wójt Gminy Szudziałowo**. |
| Zambrów, **gmina miejska** | 2014011 | 2020 | **Zambrów gmina wiejska 2014052** (w siatce, ma już osobny OK) | **TAK, miasto Zambrów (nie gmina wiejska)** | Powierzchnia **19,02 km²**, **21 112** zameldowanych; **Rada Miasta** + **Burmistrz Miasta**; **Urząd Miasta Zambrów**; MOPS, ZSM, „mieszkańców Zambrowa”. Zdanie „siedzibą … Miasta Zambrów **oraz Gminy Zambrów**” oznacza, że w mieście siedzi też UG wiejski — to nie treść gminy wiejskiej. Host `bip.zambrow.pl` (miasto); wiejska ma `ugzambrow` / siteor. |
| Pozezdrze, gmina wiejska, pow. węgorzewski | 2819022 | 2018, 2019, 2020, 2022, 2023 | brak w siatce (unikalna nazwa) | **TAK, Pozezdrze** | Szablon: „WŁADZE SAMORZĄDOWE GMINY POZEZDRZE”; **Rada Gminy Pozezdrze**; **Wójt Gminy Pozezdrze Bohdan Mohyła**; Warmińsko-Mazurskie / powiat węgorzewski (Węgorzewo, Giżycko w kontekście sąsiadów/wydarzeń). Site idcom `47007` = stały identyfikator jednostki. |
| Przasnysz, **gmina wiejska** | 1422072 | 2024 | **Przasnysz gmina miejska 1422011** (w siatce; 2024 już OK jako *raport o stanie miasta*) | **TAK, gmina wiejska (nie miasto)** | Organ: **Wójt Gminy Przasnysz**, **Rada Gminy Przasnysz**, brak burmistrza / „miasta Przasnysz”. Treść wiejska: GOPS, **Gminna Biblioteka … w Bogatem**, OSP, sołectwa. Host `przasnysz.pl` jest **mylący** (domena miasta) — rozstrzyga treść, nie host. Miasto ma osobny plik `RAPORT_O_STANIE_MIASTA_2024.pdf`. |

---

## 1b. Druga tura — 4 URL-e „strona czy plik?”

Założenie użytkownika: URL prowadzi do **strony**, trzeba wyłuskać stały URL pliku. Empiria: **GET wszystkich czterech zwraca od razu binarny dokument** (PDF/DOC), bez HTML-owego launcherа. Identyfikatory w ścieżce są **trwałe** (UUID / numeryczne id CMS), bez tokenu sesji — **nie podmieniano** URL w CSV.

| Jednostka | TERYT | Rok | Co zwraca GET | Stały URL pliku? | Werdykt tożsamości |
|---|---|---|---|---|---|
| **Wiśniew**, gmina wiejska, pow. **siedlecki** | 1426112 | 2019 | `application/pdf`, `filename=Raport_Gminy_2019.pdf`, 38,7 MB, 188 stron **skan** | Tak: `samorzad.gov.pl/attachment/{UUID}` **jest** plikiem. UUID jest niezmienny. | **TAK, Wiśniew (siedlecki), nie Wiśniewo (mławski).** OCR s. 0–4: „o stanie Gminy Wiśniew”; „WOJEWÓDZTWO MAZOWIECKIE / POWIAT SIEDLECKI / GMINA WIŚNIEW”; 5874 os. (31.12.2019), 126 km², 26 miejscowości / **27 sołectw**, „Gmina wiejska”. Brak Mławy / Wiśniewa. |
| **Grębków**, gmina wiejska, pow. węgrowski | 1433022 | 2023 | Ścieżka kończy się na **`.html`**, ale `Content-Type: application/msword` — to **`.doc`** (~0,33 MB). Z IP US: timeout; z **Tor PL**: 200. | Tak: `uggrebkow.bip.gov.pl/fobjects/download/1812513/…` — id obiektu `1812513` jest stałe. Sufiks `-doc.html` to konwencja BIP.gov.pl, nie osobna strona. | **TAK, Grębków.** „Raport o stanie gminy” + art. 28aa; 28 sołectw / 29 miejscowości / ~131 km² / 4404 os. (31.12.2023); granice **Liw, Wierzbno, Kałuszyn, Kotuń, Mokobody**; typowo wiejska. |
| **Czyże**, gmina wiejska, pow. hajnowski | 2005042 | 2018 | `application/msword`, `filename="r2.doc"`; ten sam plik pod `/download-file/id.3855` i `/id.3855/attachment.1` | Tak: id zasobu **3855** (BIT CMS) jest stałe. `/resource/3855` bez `download-file` schodzi na logowanie — **nie** używać. | **TAK, Czyże.** Tytuł „RAPORT O STANIE GMINY CZYŻE Rok 2018” (OLE gubi diakrytyki: `CZY`). Hajnowski pojawia się jako **powiat**, nie jako miasto. |
| **Hajnówka gmina wiejska** | **2005062** | 2018 | `application/msword`, `filename="raport_o_stanie_gminy.doc"`; id **16041** | Analogicznie: `bip-ughajnowka.podlaskie.eu/.../id.16041/attachment.1` jest plikiem. Prefiks hosta **`ug`** (nie `um`). | **TAK, gmina wiejska, nie miasto 2005011.** „RAPORT O STANIE GMINY HAJN… wka”; fraza **„gminy wiejskiej”**; sołectwa **Nowosady, Dubiny**; „absolutorium **wójtowi**”; Rada Gminy. Miasto ma osobny PDF 2018 na `hajnowka-bip` / siteor. |

### URL: strona vs plik — reguły

1. **Nie zgadywać po rozszerzeniu w ścieżce.** `…/raport-…-doc.html` na `*.bip.gov.pl/fobjects/download/{ID}/` często serwuje **DOC/PDF** (`Content-Type` + magic `%PDF` / OLE). Najpierw bajty, potem HTML.
2. **Podmieniać URL tylko gdy** nowy href jest **trwały**: id numeryczne CMS, UUID załącznika, `sites/{idcom}/wiadomosci/{id}/files/nazwa.pdf`. **Nie** podmieniać: `?token=`, `Expires=`, jednorazowy `getfile` z ciasteczkiem sesji, presigned S3.
3. **`samorzad.gov.pl/attachment/{uuid}`** — to już blob pliku (`Content-Disposition: filename=…`). UUID jest adresem treści; nie szukać „ładniejszej” nazwy, bo jej nie ma.
4. **BIT/podlaskie.eu:** `…/download-file/id.{N}/attachment.1` = plik. `…/resource/{N}` bywa logowaniem admina — gorszy URL.
5. **`*.bip.gov.pl`** z IP zagranicznego często **timeout/403**; ten sam URL działa z exit **PL** (Tor). To nie znaczy, że to strona HTML.
6. Host `bip-ug*` vs `bip-um*` jest **wskazówką typu** (wiejska vs miasto), nie wyrokiem (por. Przasnysz 2024 na `przasnysz.pl`).

---

## 1d. Tura „wyłuskaj bezpośredni URL pliku” (Czyże, Hajnówka wiejska, Dziadkowice, Wiśniew)

Ponowne GET 2026-08-21 (nagłówki + strony spisu BIP / samorzad.gov.pl). **Żadnego z czterech URL-i nie podmieniono** — wszystkie **już są** trwałym blobem pliku.

| Jednostka | TERYT / rok | URL w CSV | Co zwraca GET | `Content-Disposition` | Strona spisu (HTML, **nie** podmieniać) |
|---|---|---|---|---|---|
| Czyże | 2005042 / 2018 | `bip-ugczyze.podlaskie.eu/resource/file/download-file/id.3855/attachment.1` | OLE `.doc`, `application/msword`, **bez** przekierowania | `filename="r2.doc"` (18.05.2019) | [raport-o-stanie-gminy.html](https://bip-ugczyze.podlaskie.eu/raport-o-stanie-gminy.html) — pozycja „Raport o stanie Gminy Czyże rok 2018” wskazuje **ten sam** `id.3855` |
| Hajnówka wiejska | 2005062 / 2018 | `bip-ughajnowka.podlaskie.eu/resource/file/download-file/id.16041/attachment.1` | OLE `.doc`, `application/msword` | `filename="raport_o_stanie_gminy.doc"` (30.05.2019) | [raport-o-stanie-gminy-za-2018.html](https://bip-ughajnowka.podlaskie.eu/Hajnowka/raport_o_stanie_gminy/raport-o-stanie-gminy-za-2018.html) — href „Raport o stanie gminy” = **ten sam** `id.16041` |
| Dziadkowice | 2010032 / 2019 | `bip-ugdziadkowice.podlaskie.eu/resource/file/download-file/id.5296/attachment.1` | `%PDF-1.2`, `application/pdf` | `filename="zarzadzenie_nr_11820_raport.pdf"` (29.05.2020) | [raport-o-stanie-gminy.html](https://bip-ugdziadkowice.podlaskie.eu/raport-o-stanie-gminy.html) — „za rok 2019” = **ten sam** `id.5296` |
| Wiśniew | 1426112 / 2019 | `samorzad.gov.pl/attachment/c2aab080-b95b-4d0a-be9b-3b42cb3b2f77` | `%PDF-1.4`, 40 580 208 B | `filename*=UTF-8''Raport_Gminy_2019.pdf` | [raport-o-stanie-gminy2019](https://samorzad.gov.pl/web/gmina-wisniew/raport-o-stanie-gminy2019) — jedyny załącznik to **ten sam UUID** |

**Warianty, które nie są lepszym URL-em pliku:**

| Próba | Wynik |
|---|---|
| `/resource/{id}` (np. `/resource/3855`) | 302 → **logowanie admina** SmartSite (`/admin/auth/login?ref=…`) — gorszy |
| `/resource/{id}/{filename}` i `/resource/{id}/{filename}/attachment.1` (wzorzec nowszych lat: `116324/RAPORT.pdf`) | **404** dla tych starych id plików. Ładniejsza ścieżka z nazwą działa dopiero przy **id folderu/strony** (5–6 cyfr, np. 115251, 116324, 117137), nie przy id pliku BIT |
| `/resource/file/download-file/id.{N}` (bez `/attachment.1`) | ten sam blob — kosmetyka, nie nowy adres |
| `/resource/file/download-file/id.3855/r2.doc` | działa, ale `r2.doc` to śmieciowa nazwa CMS, nie poprawa trwałości |
| Hajnówka `id.16079` | **inny plik**: uchwała IX/57/19, `filename="ix5719.pdf"` (PDF-owertka). Raport 28aa to **16041** `.doc` |
| Hajnówka `id.16034` | załącznik do raportu (wykaz uchwał 2018), nie sam raport |
| Wiśniew `/web/gmina-wisniew/raport-o-stanie-gminy2019` | strona HTML spisu — **nie** podmieniać na nią URL wiersza |

**Reguła dopisana:** podmieniać URL tylko gdy nowy href jest **innym trwałym id pliku** (inny UUID / inne `id.{N}` / `resource/{folderId}/{nazwa-pliku}`) **i** GET zwraca ten sam dokument. Sufiks `/attachment.1`, kosmetyczna nazwa w ścieżce i strona spisu **nie** kwalifikują się.

---

## 1e. Tura `identity:*` (25 wierszy, 2026-08-21) — OCR 3 stron, rocznik, `.doc`, `wrong_doc_type`

Polecenie użytkownika (wiążące):

1. `identity:not_a_report` — **wszystkie to raporty**; OCR **pierwszych 3 stron**; w `metoda_pozyskania` dopisać ` + ocr`.
2. `identity:year_missing` — sprawdzić **nazwę pliku i treść** (tytuł / „za rok X”), nie sam `year_found(\\bY\\b)`.
3. `identity:gmina_missing` — wszystko to **`.doc`**, są raportami; znaleźć sposób na **poprawny odczyt DOC**.
4. `identity:wrong_doc_type` — oba pliki **się otwierają**; opisać, *dlaczego* pipeline je odrzuca (nie kasować URL).

Wynik CSV tej tury: **23 → `ok_finalnie`**, **2** zostają `not_ok` (`wrong_doc_type`). Żadnych przenosin roku (inaczej niż §1c).

### 1e.1. `identity:not_a_report` (9) → `ok_finalnie`, metoda `… + ocr`

Wymuszone OCR stron 0–2 (`tesseract pol+eng`). `looks_like_report` padało, bo warstwa tekstowa **zaczyna się od spisu treści / „Władze…” / charakterystyki**, bez frazy *raport o stanie* na s. 1. Fingerprint 28aa + gmina + rok były już w treści albo na skanie okładki.

| Jednostka | Rok | ext | Co pokazał OCR / tekst | Uwaga |
|---|---:|---|---|---|
| Przasnysz **wiejska** 1422072 | 2024 | pdf tekst 11,5 MB | s. 2: „Realizacja budżetu **Gminy Przasnysz**”; brak burmistrza | host `przasnysz.pl` mylący (domena miasta) |
| Wiśniew 1426112 | 2019 | pdf **skan** 38,7 MB | „Gmina Wiśniew”, pow. siedlecki; 188 stron, 0 znaków cyfrowych | homonim **Wiśniewo** — rozstrzyga powiat |
| Szudziałowo 2011102 | 2018 | pdf tekst 0,66 MB | „Ogólna charakterystyka gminy” + granica z Białorusią; 34 sołectwa | tytuł tylko poza warstwą / na okładce |
| Banie Mazurskie 2818012 | 2024, 2025 | pdf skan | okładka „**O STANIE GMINY BANIE MAZURSKIE**”, „STAN NA: 31 grudnia X”, „OPRACOWANO: maj X+1” | publikacja X+1 ≠ rocznik |
| Pozezdrze 2819022 | 2018, **2019 (docx)**, 2022, 2023 | pdf/docx | szablon „WŁADZE SAMORZĄDOWE GMINY POZEZDRZE”, wójt **Bohdan Mohyła** | 2019 to `.docx` — ` + ocr` i tak dopisane (polecenie dotyczyło całej klasy) |

`looks_like_report` po OCR nadal bywa `False` (np. Przasnysz: TOC bez słowa „raport”; Pozezdrze: od razu władze). **Nie wymagać frazy tytułowej**, gdy jest fingerprint 28aa.

### 1e.2. `identity:year_missing` (8) → `ok_finalnie` (rocznik z treści = wiersz)

To **nie** były pomyłki roku jak Brok/Pisz/Kobylin. Pipeline nie widział roku, bo **skan okładki / zarządzenia** nie miał warstwy tekstu. Po OCR 3 stron + `Content-Disposition` / unquote URL:

| Jednostka | Wiersz | Sygnał w treści (wygrywa) | Pułapka w nazwie / ścieżce |
|---|---:|---|---|
| Lipsk 2001043 | 2020 | art. 28aa, burmistrz, „za 2020”; plik `…2020+ROK_opt.pdf` | w URL też `2022` (folder/id) — ignorować |
| Wasilków 2002133 | 2018 | zarządzenie 31.05.**2019** „raport … **za rok 2018**” | URL `Raport_o_stanie_Gminy.pdf` **bez roku**; stary skrypt brał `2020` z id zasobu |
| Zawady 2002152 | 2022 | zarządzenie 54.**2023** z 16.06.2023 „raport … **w 2022 roku**” | filename = data **zarządzenia** (publikacja X+1), nie rocznik |
| Dziadkowice 2010032 | 2019 | zarządzenie 118/**20** (28.05.2020) + w treści 2019 | `filename="zarzadzenie_nr_11820_raport.pdf"` — `11820` to nr aktu, `20` = rok publikacji |
| Szudziałowo | 2020, 2021, 2022 | okładki „SZUDZIAŁOWO, **MAJ 2021/22/23**” = publikacja; rocznik z tytułu „za rok X” | skan — bez OCR `year_missing` |
| Nowe Piekuty 2013072 | 2021 | „RAPORT O STANIE GMINY NOWE PIEKUTY **za 2021 rok**. … **maj 2022**” | duży skan (~42 MB); z US timeout, z **Tor PL** 200 |

**Reguła empiryczna tej tury:** `za rok X` / `w X roku` / „stan na 31.12.X” w **treści lub OCR okładki** > `filename` > folder BIP. Gdy treść mówi X, a filename Y = X+1 (zarządzenie z czerwca X+1) — **zostawić X**, nie przenosić.

Priorytet odczytu roku (doprecyzowanie §3.7):

```
1. fraza tytułowa / zarządzenie: „za rok X”, „za X r.”, „w X roku”
2. „stan na 31.12.X” na okładce
3. filename / Content-Disposition, tylko gdy wygląda jak „za_XXXX” / „GMINY 2021”
4. NIE: numer zarządzenia (54.2023, 11820), id zasobu BIP, folder 2021_04, modDate
```

### 1e.3. `identity:gmina_missing` (6) → `ok_finalnie` po `catdoc`/`antiword`

Wszystko OLE `.doc`. Stary `extract_doc_ole` (`strings` cp1250 + UTF-16LE) **gubi polskie znaki** i łamie `gmina_found`:

| Token w CSV | Co widział OLE | Skutek |
|---|---|---|
| Grębków | `Gmina Gr w` | brak „Grębków” |
| Czyże | `GMINY CZY` | token 5-literowy obcięty na „że” |
| Hajnówka | `HAJN` + `wka` | kolizja z **miastem** 2005011; brak „gminy wiejskiej” |
| Mały Płock | `Gminy Ma ock` | goły „Płock” byłby kolizją z miastem; „Mały” znika |

**Działający odczyt (kolejność):**

1. `catdoc -d cp1250 -w plik.doc` — zachowuje ą/ć/ę/ł/ń/ó/ś/ź/ż;
2. `antiword -m UTF-8.txt plik.doc` — drugi przebieg UTF-8;
3. dopiero potem OLE `strings` jako fallback.

Po (1)+(2) `identity_ok` = True (gmina + rok + 28aa) dla: Grębków 2023 (`tor_pl_ip`, `uggrebkow.bip.gov.pl`), Czyże 2018, Hajnówka wiejska 2018, Mały Płock 2022/2024/2025.

LibreOffice nie był potrzebny, gdy jest `catdoc`. Fuzzy diakrytyków w `gmina_found` zostaje jako siatka bezpieczeństwa, ale **nie zastępuje** właściwego dekodera.

### 1e.4. `identity:wrong_doc_type` (2) — pliki są raportami; psuje je klasyfikator

Oba **otwierają się** i są 28aa właściwej jednostki. 2026-08-21: po opisie problemu ustawione na **`ok_finalnie`** (URL/hash bez zmian). Mechanizm, który je wcześniej zdemotował:

```
looks_like_report(text) == False   # brak frazy „raport o stanie” na s. 1
  AND
którykolwiek WRONG_DOC_HINTS w norm(text)
  → clearly_wrong_doc == True
  → identity:wrong_doc_type
```

Hinty strzelają w **rozdział / wiersz tabeli strategii**, nie w osobny dokument POŚ.

**Zambrów miasto 2020** (`bip.zambrow.pl/…/raport-o-stanie-gminy-2020.docx`, 0,13 MB)

- Treść: miasto Zambrów, **19,02 km²**, **21 112** os. (31.12.2020), **Rada Miasta + Burmistrz**, UM, MOPS, ZSM. To **2014011**, nie wiejska 2014052.
- W całym DOCX **nie ma słowa „raport”** — start: „INFORMACJE OGÓLNE”.
- Hity `WRONG_DOC_HINTS` z **tabeli programów** (dział UM / data uchwały):
  - *Gminna strategia rozwiązywania problemów społecznych na lata 2011–2020*
  - *Program ochrony środowiska Gminy Miasto Zambrów na lata 2015–2018 z perspektywą do 2022*
- Cytat **art. 30 u.s.g.** („**wójt** wykonuje uchwały rady gminy…”) tuż po zdaniu o **Burmistrzu Miasta** → `organ_signals.wojt=True` (fałsz). To nie jest gmina wiejska.

**Pozezdrze 2020** (`idcom …/572843/files/raport.pdf`, 1,65 MB)

- **Ten sam szablon** co 2018/2019/2022/2023 (te są już `ok_finalnie`): „Władze samorządowe Gminy Pozezdrze”, wójt **Bohdan Mohyła**, Rada Gminy, UG.
- Znowu brak frazy tytułowej na s. 1.
- Hint: rozdział *Strategia rozwiązywania problemów społecznych w Gminie Pozezdrze*.
- `prezydent=True` przez *„Medal … przyznawane przez **Prezydenta RP**”* — nie prezydent miasta.

**Wniosek do pipeline:** `WRONG_DOC_HINTS` wolno odpalać tylko gdy dokument **nie** ma fingerprintu 28aa (rada + organ + 31.12.Y / budżet). Tytuł nie jest konieczny. Cytat art. 30 + Burmistrz ≠ wójt. „Prezydent RP” ignorować.

---

## 1c. Tura „zły rok” — raport za X wisiał na wierszu Y ≠ X

Trzy URL-e **były** raportami 28aa właściwej gminy, ale **rocznik w treści ≠ `rok_raportu` wiersza**. W siatce istniał wiersz na rok z dokumentu (status `BRAK`) — URL przeniesiono tam, zły wiersz wyczyszczono.

| Wiersz (źle) | Co jest w dokumencie | Publikacja / ścieżka | Właściwy wiersz |
|---|---|---|---|
| Brok-**2024** | „RAPORT O STANIE GMINY BROK **ZA 2022 R.** … Brok, **maj 2023**” | nazwa pliku `…za 2022 rok.pdf` | Brok-**2022** → `ok_finalnie` |
| Pisz-**2025** | „RAPORT O STANIE GMINY PISZ **ZA ROK 2022** **Maj 2023**”; `filename="RAPORT O STANIE GMINY 2022.pdf"` | `download.php?id=25070` (id **starsze** niż raport 2024 id=27942) | Pisz-**2022** → `ok_finalnie` |
| Kobylin-Borzymy-**2020** | „RAPORT O STANIE GMINY KOBYLIN – BORZYMY **ZA 2018 ROK**” | folder BIP `2021_04/` (kwiecień **roku publikacji**, nie roku raportu; i tak X+1 byłoby 2019 — tu plik wrzucono jeszcze później) | Kobylin-Borzymy-**2018** → `ok_finalnie` |

`year_found(Y)` na złym wierszu było **false** (stąd `identity:year_missing`) — to nie „brak roku w PDF”, tylko **inny** rok niż kolumna.

---

## 2. Dlaczego automat je odrzucił

Wspólny mechanizm (tura 1): **okładka zeskanowana albo jej nie ma**, a silnik wymaga frazy „raport o stanie gminy/miasta” w wyciągniętym tekście.

| Przypadek | `failure_reason` | Przyczyna techniczna |
|---|---|---|
| Pozezdrze, Szudziałowo, Przasnysz wiejska | `not_a_report` | PDF zaczyna się od „Władze…” / „Charakterystyka…” / spisu treści. Frazy tytułowej **nie ma w warstwie tekstowej**. |
| Mały Płock | `gmina_missing` | OLE `.doc` (cp1250): „ł” ginie → „Gminy Ma **ock**”. Token „Płock” sam byłby kolizją z miastem. |
| Zambrów miasto | `wrong_doc_type` | Brak frazy tytułowej **oraz** wiersz „Program ochrony środowiska…” w tabeli strategii. Cytat art. 30 ustawia fałszywą flagę `wojt`. |
| Wiśniew 2019 | `not_a_report` | 188 stron **czystego skanu** (0 znaków/strona). Bez OCR tytuł („o stanie Gminy Wiśniew”) nie istnieje. Pipeline OCR-ował za słabo / wcale przy `chars_per_page < 60` na całym pliku 40 MB. |
| Grębków 2023 | `gmina_missing` | OLE: „Grębków” → „Gmina **Gr w**”; sołectw → „so ectw”. `gmina_found` pada na diakrytykach mimo art. 28aa w treści. Dodatkowo timeout `bip.gov.pl` z IP US. |
| Czyże 2018 | `gmina_missing` | OLE: „Czyże” → „GMINY **CZY**”. Krótki token + utrata „że”. |
| Hajnówka wiejska 2018 | `gmina_missing` | OLE: „Hajnówka” pęka na „HAJN” + „wka”; „wójtowi” → „w jtowi”. Nazwa koliduje z **miastem** Hajnówka — sam token „Hajnówka” nie rozstrzyga, a „gminy wiejskiej” / Nowosady / Dubiny nie były użyte. |

`year_found` często **był true**. Odrzucenie szło z warstwy „czy to raport” / „czy to ta jednostka” / ekstrakcji `.doc`/skanu.

---

## 3. Jak lepiej rozpoznawać raport o stanie gminy

### 3.1. Nie opierać decyzji wyłącznie na frazie tytułowej

Fraza *„raport o stanie gminy”* / *„raport o stanie miasta”* jest mocnym sygnałem, ale:

- często jest **tylko na skanie okładki** (Wiśniew: s. 0 „o stanie / Gminy Wiśniew”; OCR 5 stron **obowiązkowo**, gdy strony 0–4 mają ~0 znaków);
- wiele urzędów **nie wstawia tytułu** do warstwy tekstu (Pozezdrze, Szudziałowo, Przasnysz wiejska).

**Fingerprint strukturalny art. 28aa** (wystarczy 3+ z listy, plus organ+geografia):

1. Organ stanowiący: *Rada Gminy X* **albo** *Rada Miasta X*.
2. Organ wykonawczy zgodny z typem: **wójt** / **burmistrz** / **prezydent**.
3. Budżet **za rok Y** albo stan ludności **na 31.12.Y**.
4. Typowe działy: inwestycje, oświata, pomoc społeczna (GOPS/MOPS), odpady, drogi, OSP, ludność.
5. Odniesienie do art. 28aa / „do 31 maja” / debata / absolutorium.
6. Charakterystyka: powierzchnia, liczba sołectw **albo** osiedli, siedziba urzędu.

To jest raport nawet bez słów „raport o stanie” na stronie 1. Przykład Wiśniew: OCR dał tylko „o stanie Gminy Wiśniew” + „Gmina wiejska” + 27 sołectw + 31.12.2019 — to wystarczy.

### 3.2. Kolizje nazw: organ + geografia + TERYT, nie sam token nazwy

| Wzorzec | Przykład | Reguła |
|---|---|---|
| Miasto vs gmina wiejska o tej samej nazwie | Przasnysz 1422011 vs 1422072; Zambrów 2014011 vs 2014052; **Hajnówka 2005011 vs 2005062** | **Rozstrzyga organ i instytucje.** Miasto: Rada Miasta, burmistrz/prezydent, UM, MOPS. Wiejska: Rada Gminy, wójt, UG, GOPS, **sołectwa** (Hajnówka: Nowosady, Dubiny; fraza „gminy wiejskiej”). Host `bip-ug*` vs `bip-um*` / `hajnowka-bip` jako wskazówka. |
| Token zawarty w innej nazwie | Mały **Płock** vs Płock | Nie matchować samego „Płock”. Pełna nazwa / odmiana **albo** powiat kolneński + podlaskie + SIMC (Chludnie, Korzeniste). |
| Homonim **innej gminy** (nie pary miasto/wiejska) | **Wiśniew** (siedlecki 1426112) vs **Wiśniewo** (mławski 1413102) | Wymagać powiatu **siedlecki** / woj. mazowieckie / sołectw Wiśniewa. Samo „Wiśniew” w OCR jest prefiksem „Wiśniewo” — **nie** akceptować prefiksu bez granicy słowa / bez powiatu. |
| Unikalna nazwa | Pozezdrze, Szudziałowo, Czyże, Grębków | Nazwa + wójt/28aa + województwo; idcom `site=` / BIP id zasobu jako klucz. |

**Fałszywe flagi organu — ignorować:**

- *„wójt”* w cytacie **ustawy** (art. 30) przy *Burmistrz Miasta* → nie wiejska.
- *„Prezydent RP”* (medale) → nie prezydent miasta.
- Sąsiad / powiat / starosta jako partner (Zambrów o „Gminie Zambrów”; Pozezdrze o Węgorzewie; Czyże o powiecie hajnowskim) → nie zmienia autora.
- OLE: `w jtowi` = **wójtowi** (absolutorium) — to **jest** sygnał wójta, nie śmieć.

**m.n.p.p. vs powiat ziemski:** TERYT 7-znakowy + typ organu; inwestycje UM m.n.p.p. nie wchodzą w powiat ziemski.

### 3.3. Sygnały z URL i BIP — priorytet, nie wyrok

| Sygnał | Siła | Uwaga |
|---|---|---|
| Host `bip-ug{gmina}.podlaskie.eu` / `bip-um{gmina}` | wysoka | `ug` ≈ wiejska (`ughajnowka`, `ugczyze`), `um` ≈ miasto; treść i tak sprawdzić. |
| idcom `sites/{SITE_ID}/` | wysoka | `47007` = Pozezdrze; mapa site→TERYT. |
| `*.bip.gov.pl/fobjects/download/{ID}/` | wysoka | `{ID}` trwałe; `.html` w slugu **nie** znaczy HTML. Często wymaga IP PL. |
| Nazwa pliku `Raport+o+stanie+Gminy+X+za+YYYY` | średnia+ | Dobry prior; nie zastępuje organu przy homonimach. |
| Domena urzędu miasta (`przasnysz.pl`, `bip.zambrow.pl`) | **niska sama** | Gmina wiejska bywa na domenie miasta. Treść > host. |
| `samorzad.gov.pl/attachment/uuid` | średnia jako **adres pliku** | To blob, nie strona. Tożsamość **tylko z treści** (Wiśniew: OCR). |

### 3.4. Ekstrakcja tekstu

1. **Pełny skan (Wiśniew):** 188 stron, 0 znaków cyfrowych. OCR **pierwszych 5 stron** jest warunkiem koniecznym; `looks_like_report` musi akceptować „o stanie Gminy X” bez słowa „raport” na tej samej linii, jeśli reszta fingerprintu 28aa jest.
2. **PDF mieszany:** OCR każdej z pierwszych 5 stron z &lt; ~60 znakami, nawet gdy dalsze strony są `tekst`.
3. **OLE `.doc`:** nie ufać `strings`/OLE. **Najpierw** `catdoc -d cp1250 -w`, potem `antiword -m UTF-8.txt`, OLE dopiero jako fallback. Empiria §1e.3: po `catdoc` `identity_ok` przechodzi (Grębków, Czyże, Hajnówka wiejska, Mały Płock). Surowe OLE gubi diakrytyki: Grębków→`Gr w`, Czyże→`CZY`, Hajnówka→`HAJN`/`wka`, Mały Płock→`Ma ock`, wójt→`w jt`. Fuzzy (`Gr[eę]?b?k[oó]w`, `Czy[zż]e`, `Hajn[oó]wk?a`, `w[ .]?jt`) zostaje siatką, nie zastępuje dekodera.
4. **`WRONG_DOC_HINTS`:** nie karać POŚ / „strategii rozwiązywania problemów” jako **rozdziału / wiersza tabeli** raportu 28aa (Zambrów 2020, Pozezdrze 2020 — §1e.4). Odpalać hint tylko gdy brak fingerprintu 28aa.
5. Cytaty ustaw wyłączyć z detekcji organu (art. 30 „wójt” przy Burmistrzu; „Prezydent RP”).
6. **OCR:** dla klasy `not_a_report` — **zawsze pierwsze 3 strony** (`pol+eng`), nawet gdy PDF ma warstwę tekstu (tytuł bywa tylko na skanie okładki). W `metoda_pozyskania` dopisać ` + ocr`.

### 3.5. Minimalna wiązka dowodowa

Dokument = raport **tej** gminy za **ten** rok, gdy:

```
(A) fingerprint 28aa  AND
(B) rok Y w treści / metadanych / „stan na 31.12.Y”  AND
(C) jednostka:
      TERYT w treści
      OR (organ zgodny z typem AND (pełna nazwa w odmianie OR ≥2 sołectwa/SIMC
          OR (powiat + województwo + nazwa z granicą słowa)))
      AND NOT (organ sprzeczny; homonim bez powiatu — Wiśniew/Wiśniewo;
               prefiks nazwy bez \b)
```

Sama para *nazwa + rok w URL* **nie wystarcza**. Sama fraza tytułowa **nie jest konieczna**, jeśli (A)+(C) są mocne.

**(B) jest obowiązkowe i dotyczy roku merytorycznego, nie daty pliku.** Samo wystąpienie cyfr `2024` w budżecie/strategii nie czyni dokumentu raportem za 2024, jeśli tytuł mówi „za 2022 r.”.

### 3.6. Checklist kolizji przy homonimach

Pary w siatce m.in.: Przasnysz, Zambrów, Łomża, Kolno, Hajnówka, (poza siatką: Płock), plus **Wiśniew ≠ Wiśniewo**.

- [ ] „Wójt Gminy X” vs „Burmistrz / Prezydent Miasta X”
- [ ] „Rada Gminy” vs „Rada Miasta”
- [ ] „Urząd Gminy” vs „Urząd Miasta”
- [ ] GOPS vs MOPS
- [ ] sołectwa / lista wsi vs osiedla / ulice
- [ ] powierzchnia i liczba ludności (rząd wielkości)
- [ ] siedziba: „miejscowość X” vs „miasto X”
- [ ] fraza „gminy wiejskiej” (Hajnówka 2018)
- [ ] powiat w nagłówku/okładce (Wiśniew: Siedlecki ≠ Mławski)
- [ ] drugi TERYT 7-znakowy — jeśli jest i nie nasz → odrzucić
- [ ] host BIP (`ug` / `um`) jako wskazówka, nie wyrok
- [ ] `\b` przy nazwie, która jest prefiksem innej (Wiśniew / Wiśniewo)

### 3.7. Rocznik — rok X w raporcie, publikacja w roku X+1 (obowiązkowe)

Kolumna `rok_raportu` = rok **którego dotyczy** sprawozdanie (stan na 31.12.X, budżet roku X), **nie** rok wgrania na BIP i **nie** data na okładce „maj / czerwiec X+1”.

Ustawa: organ wykonawczy przedstawia radzie raport za rok X **do 31 maja roku X+1**. Typowy wzorzec tytułu:

> Raport o stanie gminy … **za 2022 r.**  
> Gmina …, **maj 2023 r.**

| Źródło roku | Siła | Pułapka |
|---|---|---|
| Tytuł / okładka: **„za rok X”**, **„za X r.”**, **„za X rok”** | **najwyższa** | to jest rocznik merytoryczny |
| `filename` / `Content-Disposition`: `…za_2022…`, `RAPORT … 2022.pdf` | wysoka | slug `-doc.html` i folder `2021_04` to **nie** rok X |
| „stan na **31.12.X**”, „budżet gminy na rok X”, „wykonanie budżetu X” | wysoka, gdy spójne z tytułem | w raporcie za X naturalnie występują też X−1 (porównania) i X+1 (WPF, „lata 2016–2022”) |
| Data na dole okładki: „maj X+1”, „czerwiec X+1” | **publikacja**, nie rocznik | **nie** wpisywać jako `rok_raportu` |
| Folder BIP `2021_04`, `2025_06`, data pliku, `modDate` PDF, id `download.php?id=` | publikacja / CMS | Brok 2024←2022; Pisz id=25070 przy wierszu 2025, treść 2022; Kobylin `2021_04` przy raporcie **2018** |
| Numer zarządzenia w filename (`54.2023`, `zarzadzenie_nr_11820_raport.pdf`) | **publikacja / nr aktu** | Zawady 2022: zarządzenie 16.06.**2023**; Dziadkowice 2019: `11820` = nr 118/20 |
| Samo `year_found(Y)` (regex `\bY\b` gdziekolwiek) | **za słabe** | raport za 2022 zawiera 2019–2023 w tabelach; nie dowodzi, że to raport za 2024 |

**Reguły:**

1. **Potwierdzić rocznik z treści** (tytuł / pierwsze strony / OCR okładki) **zanim** `ok_finalnie`. Jeśli tytuł mówi „za X”, a wiersz ma Y ≠ X → **nie** akceptować na Y.
2. **Publikacja X+1 jest normą**, nie błędem. „Maj 2023” + „za 2022” = wiersz **2022**.
3. Gdy Y ≠ X i w siatce **jest** wiersz gmina+X: **przenieść URL** na X (po weryfikacji tożsamości), wiersz Y wyczyścić (`BRAK`). Nie trzymać tego samego pliku na dwóch latach.
4. Gdy Y ≠ X i **brak** wiersza X w siatce: wyczyścić Y; nie „przepinać” na sąsiedni rok na ślepo.
5. **Nie** brać roku z samego URL-a (`…/2025_02/…`, `raport2021.pdf` przy wierszu 2020 — to bywa publikacja albo pomyłka nazwy). Treść > ścieżka > folder.
6. Konflikt sygnałów (tytuł „za 2018”, w budżecie głównie 2019): wygrywa **formuła „za rok …”** na stronie tytułowej / w nagłówku art. 28aa.
7. PowerBI / raport interaktywny: ten sam wymóg — rok **w tytule dashboardu / pierwszej stronie**, nie rok linku.

Checklist rocznika:

- [ ] Jest fraza **„za rok X” / „za X r.” / „za X rok”** (treść lub OCR okładki)
- [ ] X = `rok_raportu` wiersza
- [ ] Data „miesiąc X+1” potraktowana jako publikacja
- [ ] Folder BIP / `modDate` / id download **nie** nadpisują X
- [ ] Jeśli X ≠ wiersz i istnieje wiersz X → przenieś, nie duplikuj

---

## 4. Rekomendacje do pipeline

1. `looks_like_report`: fingerprint 28aa; akceptować „o stanie gminy X” bez „raport” na okładce skanu.
2. OCR pierwszych 5 stron, jeśli **któraś** jest uboga w tekst — także gdy cały PDF jest skanem 40 MB (nie rezygnować).
3. `clearly_wrong_doc`: nie odpalać na POŚ/strategii, gdy jest budżet i rada.
4. `organ_signals`: cytaty ustawy; „Prezydent RP”; OLE `w jt` = wójt; art. 30 + Burmistrz ≠ wiejska.
5. `gmina_found`: fuzzy diakrytyków; **zakaz** gołego drugiego członu (Płock) i **zakaz** prefiksu bez `\b` (Wiśniew⊂Wiśniewo); backup SIMC/sołectw + powiat.
6. Słownik `idcom site_id → TERYT`, `bip-ug*|bip-um* → typ`, `fobjects/download/{id}` jako stały plik.
7. Download: magic bytes przed klasyfikacją HTML; `*.bip.gov.pl` przez `tor_pl_ip` przy timeout/403.
8. Podmiana URL tylko na trwały id (UUID, id CMS, idcom files); nigdy sesja/token.
9. Przy parze miasto/wiejska: twardy organ (pkt 3.6) zanim `gmina_found` na gołej nazwie.

---

## 5. Stan w CSV po weryfikacji

- §1 + §1b: **15** wierszy `ok_finalnie` (hash, `file_size_MB`, `dt_utc_finalne`; Grębków: `tor_pl_ip`).
- §1c: Brok-2024, Pisz-2025, Kobylin-Borzymy-2020 **wyczyszczone**; URL na Brok-**2022**, Pisz-**2022**, Kobylin-Borzymy-**2018**.
- §1d: Czyże-2018, Hajnówka wiejska-2018, Dziadkowice-2019, Wiśniew-2019 — URL **bez zmian** (już były plikiem).
- §1e: **23** z 25 `identity:*` → `ok_finalnie` (9× ` + ocr`; 8× year potwierdzony treścią; 6× `.doc` przez `catdoc`/`antiword`). **2** zostają `not_ok` / `identity:wrong_doc_type` (Zambrów miasto 2020, Pozezdrze 2020) — URL zachowany; to fałszywy hint POŚ/SRPS.

JSON tury 1: `/home/user/work/probe_identity_11.json`.

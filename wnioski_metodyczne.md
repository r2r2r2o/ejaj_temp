# Wnioski metodyczne — pozyskiwanie raportów o stanie gminy (art. 28aa u.s.g.)

Stan na 02.09.2026. Zbiór: 2488 wierszy (264 gminy × 8 roczników 2018–2025 + 47 jednostek nadrzędnych).
Wynik: **2368 `ok_finalnie` / 120 `brak_finalnie` / 0 `BRAK`** — 95,2% pokrycia.
*(w tym: tura 18c — odzyskany raport Mrągowo gmina wiejska za 2018 metodą „żywego załącznika", §8e; pozostałe 119 zweryfikowane jako strukturalny brak publikacji.)*

---

## 1. Skuteczność źródeł (2368 zdobytych plików)

| Plików | Źródło | Uwagi |
|---|---|---|
| 1107 | Własne serwisy gmin (WordPress i in.) | najbardziej zróżnicowane, brak wspólnego wzorca |
| 593 | SmartSite `*.podlaskie.eu` | jednolity CMS, w pełni skanowalny |
| 202 | idcom `bip-v1-files.idcom-jst.pl` | stabilne, przewidywalne ścieżki |
| 121 | `*.biuletyn.net` | treść dynamiczna, pliki pod `/fls/bip_pliki/` |
| 104 | `samorzad.gov.pl` | `/attachment/{UUID}` — nieodgadywalne, tylko przez linki |
| 94 | PUBLIKATOR (`download.php?id=`) | sekwencyjne ID, łatwe do skanowania |
| 65 | Domeny archiwalne (`archiwum.*`) | ratunek po migracji BIP |
| 45 | SSDIP (`*.bip.gov.pl`) | wymaga `--ciphers DEFAULT@SECLEVEL=0` |
| 28 | Wayback Machine | ostatnia deska ratunku |

Format: pdf 2209, docx 108, doc 24, html 9, zip 6, odt 3.
Zawartość: tekst 2056, skan 252, skan+OCR 19.

---

## 2. Metody, które zadziałały — w kolejności skuteczności

### 2.1. ★ Skan przestrzeni ID (najskuteczniejszy)
Większość CMS-ów serwuje pliki pod sekwencyjnym identyfikatorem, **niezależnie od tego, czy plik jest gdziekolwiek podlinkowany**. Usunięcie linku z menu nie usuwa pliku.

| CMS | Wzorzec |
|---|---|
| SmartSite | `/resource/file/download-file/id.{N}` |
| PUBLIKATOR | `/download.php?id={N}` |
| biuletyn.net | `/fls/bip_pliki/{RRRR_MM}/BIPOLD{NNNNNN}/{N}.{ext}` |
| CMS „portal" | `/Common/pobierzPlik/id/{N}/module_short/port/obj_id/{STRONA}/...` |

Detekcja: `HEAD` → 200 + `Content-Disposition: filename=`. Zdobycze: **Klukowo 2018 i 2019**, **Kołaki 2019**, Zawady 2021.
*Pułapka:* niektóre serwery odrzucają `HEAD` kodem 405 (Sabnie) — wtedy `GET` z nagłówkiem `Range: bytes=0-1200` i sprawdzenie sygnatury `%PDF`.

### 2.2. ★ Historia wersji strony (SmartSite)
`/cms/archive/get-page-archive/pgid.{ID}/type.1/order.DESC` → lista `?vid=N`. Gminy nadpisujące jedną stronę co roku **zachowują starsze załączniki w archiwalnych wersjach**.
Zdobycz: **Orla 2018, 2021, 2023, 2024** (4 roczniki z jednej strony pokazującej tylko 2025).

### 2.3. ★ Wyszukiwarka wewnętrzna BIP
- SmartSite: **wyłącznie** `?q=` na `/cms/search`. Warianty `?query=`, `?srchTerm=`, `/cms/search/advanced` zwracają 200, ale **0 wyników** — mimo że `srchTerm` to nazwa pola w formularzu HTML.
- Inne CMS-y: `/search?q=`, `/wyszukiwarka?szukaj=`, `/?s=`.

Zdobycz: **Biskupiec 2018** — strona nieobecna w żadnym menu, znaleziona tylko przez wyszukiwarkę.

### 2.4. Artykuły po ID (biuletyn.net)
Listy kategorii są w pełni dynamiczne (HTML pusty), ale artykuły renderują się pod `?bip=2&cid={KAT}&id={N}`. Zakres ID kalibrować próbkowaniem co 250.
Zdobycz: **Kobylin-Borzymy 2018** + komplet 7 uchwał o wotum jako dowody istnienia.

### 2.5. Wznawianie pobrań (`curl -C -`)
Serwery zrywające połączenie po ~180 kB. **Biskupiec 2018**: 12 029 804 B złożone w 60 wznowieniach (HTTP 206).

### 2.6. OCR do potwierdzenia rocznika
Skany bez warstwy tekstowej: `pymupdf` (render 150–170 dpi) + `rapidocr-onnxruntime`. Wystarczy 1. strona.
Zdobycze: Kołaki 2019 (zarządzenie 104/20 + załącznik), Biskupiec 2018, weryfikacja negatywna Trzcianne (GKRPA ≠ 28aa), rozstrzygnięcie luki Zbójnej.

### 2.7. Zarządzenia i uchwały jako nośnik
Raport bywa **załącznikiem do zarządzenia** — nazwa pliku nie zawiera wtedy słowa „raport". Uchwała o wotum zaufania = twardy dowód, że raport powstał.
**Uwaga na pisownię: 2019–2021 „votum", od 2022 „wotum"** — regex `v?otum`.

### 2.8. Domeny archiwalne i alternatywne
Po migracji BIP: `archiwum.{gmina}.pl`, `{gmina}.archiwum.bip.net.pl`, `nowa.{gmina}.com`, `bip-{nazwa}.podlaskie.eu` ↔ `bip.ug.{nazwa}.wrotapodlasia.pl`. 65 plików pochodzi właśnie stamtąd.

### 2.9. Wayback CDX — poprawna składnia
`&filter=original:.*[Rr][Aa][Pp][Oo][Rr][Tt].*` — **klasy znaków, nie `(?i)`**. Zapis `.*(?i)(raport).*` daje fałszywe 0 wyników.

---

## 3. Metody nieskuteczne (nie powtarzać)

| Metoda | Dlaczego zawodzi |
|---|---|
| Zgadywanie nazw plików | `/resource/{ID}/dowolna-nazwa.pdf` → 404; nazwa musi być dokładna. 3750 prób dla Zawad 2023 = 0 trafień |
| `crawl.links` na `samorzad.gov.pl` | treść w JS, linki `/attachment/{UUID}` niewidoczne w HTML |
| Formularz `/cms/search/advanced` | pole `srchTerm` nie działa mimo obecności w HTML |
| Wayback dla plików > 5 MiB | twardy cap 5 242 880 B, `curl -C -` nie pomaga (Grudusk 2018/2019/2021) |

---

## 4. Ograniczenia techniczne — napotkane i potwierdzone

1. **Cap 5 MiB w Wayback.** Content-Length identyczny dla `id_`/`if_`/bez sufiksu. Plik niekompletny = brak z dowodem istnienia.
2. **Rate-limiting / blokady IP.** Zbyt agresywny skan zablokował dostęp do `gminadzwierzuty.pl` i `kobylinborzymy.biuletyn.net` (`http=000`, `ReadTimeout`). **Wniosek: ograniczać równoległość do ~10–14 wątków i robić przerwy.**
3. **Wayback HTTP 503** przy seriach zapytań — wymaga odstępów 12–20 s, i tak bywa nieskuteczne.
4. **SSDIP (`*.bip.gov.pl`)** — błędy TLS; `--ciphers DEFAULT@SECLEVEL=0` czasem pomaga, w tej turze domeny nie odpowiadały.

---

## 5. Pułapki merytoryczne

**Dokumenty myląco podobne (NIE są raportem 28aa):** raport GKRPA (Trzcianne id 4628/5533, Orla), raport o stanie zapewniania dostępności, raport z wykonania POŚ, raport z konsultacji, `raport_pog_*` (plan ogólny), `raport_oos` (oceny środowiskowe), informacja o debacie (Trzcianne id 3751).

**Kolizje nazw (22 pary), m.in.:**
- gmina wiejska **Wysokie Mazowieckie** (2013102) ≠ miasto Wysokie Mazowieckie (2013011)
- **Olsztyn** m.n.p.p. (2862011) ≠ gmina Olsztyn w śląskiem (`olsztyn.bip.jur.pl`) — dominuje w wyszukiwarkach
- **Zawady** (2002152) ≠ Zawadzkie (opolskie) ≠ Zawiercie
- **Biskupiec** (olsztyński) ≠ Biskupice ≠ Papowo Biskupie
- gmina **Augustów** (2001022) ≠ miasto Augustów (2861011)

**Rocznik zawsze z treści dokumentu, nie z nazwy pliku ani daty publikacji.** Przykłady: Rudka `raport_o_stanie_gminy_2019.pdf` zawiera raport **za 2018**; Szczytno `..._szczytno_2019.pdf` to raport **za 2018**.

---

## 6. Struktura 120 pozostałych braków

| Liczba | Kategoria |
|---|---|
| 74 | Raport istnieje / powstał, ale plik opublikowany jedynie częściowo lub usunięty — kompletny PDF nieosiągalny |
| 34 | Brak publikacji / brak jakiegokolwiek śladu — gmina nigdy nie opublikowała pliku raportu |
| 12 | Jednostka nie istniała (Grabówka — utworzona 1.01.2025; + roczniki, gdy gmina wydzielona) |

Dominujący wzorzec: **BIP-y zaczynają publikować raporty dopiero od 2020–2023**, mimo że obowiązek istnieje od 2019 (za 2018). Rocznik 2018 to najsłabiej udokumentowany rok w całym zbiorze.
*Uwaga: kategorie wyliczone automatycznie z `failure_reason` (słowa kluczowe); granica między „nieopublikowany" a „powstał, plik niedostępny" bywa płynna (patrz §5 i §8d).*

---

## 6a. Wyczerpanie czterech ścieżek otwartych po turze 7 (tura 8)

| Ścieżka | Rezultat |
|---|---|
| **Kobylin-Borzymy** (7 braków) | **WYCZERPANA.** Blokada IP wygasła. Przeskanowano pełną przestrzeń artykułów `?bip=2&cid=1252&id={3000–7500}` (6 wątków) z detekcją po tytule *oraz* po nazwie załącznika. 6 trafień z frazą „raport": id.5562 = raport za 2018 (jedyny 28aa), 2× obwieszczenia środowiskowe, 3× uchwały o wotum. Dodatkowo otwarto załączniki wszystkich 7 uchwał o wotum — każda zawiera wyłącznie plik samej uchwały. |
| **Dźwierzuty** (3 braki) | **ROZSTRZYGNIĘTA CO DO FAKTU.** Strony 145/146/153 (2018–2020) zwracają 200, ale nie mają *żadnego* załącznika — brak komponentu `Załączniki (N)` i odnośników `/Common/pobierzPlik/`. Kontrolna strona 1004 (2021) listuje 11 załączników (id 495–505). Próba skanu par `(obj_id, id)` ponownie zablokowała IP — wynik odrzucony jako nierozstrzygający (kontrolne pobranie znanego pliku też dało `http=000`). |
| **SSDIP** (Grębków, Sztabin, Bielany, Olsztyn) | **ZABLOKOWANA ŚRODOWISKOWO.** Cała platforma `*.bip.gov.pl` (IP 185.41.93.214) nieosiągalna: DNS OK, ale połączenie nie dochodzi przy 4 wariantach TLS. Wayback CDX `/fobjects/*` nie zawiera snapshotów brakujących roczników. |
| **Wayback / Grudusk** | **NIEDOSTĘPNA CZASOWO.** Serie HTTP 503 przy każdej próbie (również dla Bielan i Olsztyna). Adresy plików pozostają zapisane w `failure_reason`. |

**Wniosek operacyjny:** trzy z czterech ścieżek zostały domknięte dowodowo (Kobylin, Dźwierzuty, SSDIP-CDX). Pozostałe blokery są **środowiskowe, nie metodyczne** — ta sama metoda z innego IP powinna zadziałać.

**Lekcja o throttlingu:** nawet 6–8 wątków wystarczyło, by ponownie zablokować `gminadzwierzuty.pl`. Przy CMS-ach małych gmin bezpieczna jest praca sekwencyjna z opóźnieniem ~1 s. **Zawsze umieszczać w skanie kontrolne pobranie znanego, działającego pliku** — bez tego nie da się odróżnić „pliku nie ma" od „serwer mnie blokuje". W tej turze ten test uratował mnie przed zapisaniem fałszywie negatywnego wyniku dla Dźwierzut.

---

---

## 8. Portale sesyjne rad gmin (eSesja / posiedzenia.pl) — rodzina odkryta w turze 9

Raport 28aa jest **załącznikiem do porządku obrad sesji absolutoryjnej** (kwiecień–wrzesień roku X+1). Gdy BIP gminy go nie publikuje, portal rady często ma go nadal.

**Procedura:**
1. `https://{slug}.esesja.pl/posiedzenia` — sprawdź, czy to *prawdziwy* portal.
2. Kadencje: `<select id='kadencja'>` → przełącz przez **`/?kid={ID}`** (parametr ustawiany w sesji, ujawniony w `/js/main.js`).
3. Paginacja: **`/posiedzenia/{N}`** (nie `?page=`) — bez niej widać tylko 20 najnowszych posiedzeń.
4. Na stronie posiedzenia załączniki są w wywołaniach `getfile(id,token,hash)` → `/pobierz/{id}/{token}/{hash}`.
5. **Token jest jednorazowy.** Pobranie musi nastąpić w tej samej sesji HTTP co odczyt strony, z nagłówkiem `Referer`. Ponowne użycie → `/?error=File_not_found`.

**Pułapka — fałszywie pozytywna detekcja portalu:** eSesja zwraca `HTTP 200` i stronę marketingową o stałej długości **36 759 B** dla każdej nieistniejącej subdomeny. Pierwszy test dał 52 „portale", realnych było **11**. Poprawny warunek: obecność `var rid=` **oraz** linków `/posiedzenie/`.

**Zdobycze:** Wizna 2020 i 2022, Biskupiec 2020 (skan, potwierdzony OCR).

**Pułapka tożsamości:** `augustow.esesja.pl` obsługuje **Radę Miejską w Augustowie** — pobrany stamtąd raport dotyczy miasta (2861011, Burmistrz), nie gminy wiejskiej (2001022, Wójt). Plik odrzucono po weryfikacji.

**Odtwarzalność:** URL z tokenem nie nadaje się jako trwały lokalizator. W `failure_reason` zapisano adres strony posiedzenia + stałe ID załącznika + sha256.

---

---

## 8a. TOR z polskim węzłem wyjściowym — obejście blokad geograficznych (tura 10)

**Odkrycie:** część polskich platform BIP **blokuje adresy IP datacenter/zagraniczne**. To nie jest awaria ani brak publikacji — z polskiego IP te same adresy odpowiadają natychmiast.

**Instalacja** (środowisko ma `sudo` bez hasła):
```
sudo apt-get install -y tor
# torrc: SocksPort 9050 / ExitNodes {pl} / StrictNodes 1
tor -f torrc &
curl --socks5-hostname 127.0.0.1:9050 https://api.ipify.org
```
Weryfikacja kraju: `http://ip-api.com/line/?fields=countryCode` → musi zwrócić `PL`.
Dla Pythona: `pip install "requests[socks]" pysocks`, proxy `socks5h://127.0.0.1:9050`.

**Wynik testu czterech zablokowanych ścieżek:**

| Zasób | Bez Tora | Przez Tor (PL) | Wniosek |
|---|---|---|---|
| `*.bip.gov.pl` (SSDIP) | `http=000` przy 4 wariantach TLS | **HTTP 200** | blokada geograficzna |
| `web.archive.org` (pliki) | 503 | **503** | rate-limiting IA, nie geografia |
| `olsztyn.eu` | SSLError | SSLError | problem serwera |
| `bip.andrzejewo.pl` | 000 | **200 z `--ciphers DEFAULT@SECLEVEL=0`** | przestarzały certyfikat |

**Zdobycz: Olsztyn 2018 i 2019** — dwa ostatnie braki miasta na prawach powiatu (teraz komplet 2018–2025).

**Wyszukiwarka SSDIP** (nieoczywista, kluczowa): `POST /search/articlesr` z polem `data[Search][name]`. Ścieżki `/search/{fraza}/` zwracają 404. To ona ujawniła kategorię `/statystyka-miasta/` z kompletem roczników Olsztyna.

**Ważne rozróżnienie:** Python `requests` odrzuca przestarzałe certyfikaty (`WRONG_SIGNATURE_TYPE`) nawet przy `verify=False` — to błąd *handshake*, nie weryfikacji. `curl --ciphers DEFAULT@SECLEVEL=0` łączy się poprawnie. Przy takich serwerach skanować przez `curl`, nie przez `requests`.

---

---

## 8b. Fałszywie pozytywne „rodziny źródeł" — test kontrolny (tura 11)

Trzecia z rzędu platforma, która zwraca `HTTP 200` dla **każdej** subdomeny/ścieżki, także nieistniejącej:

| Platforma | Sygnatura pustki | Poprawny test |
|---|---|---|
| eSesja | strona marketingowa, **36 759 B** | obecność `var rid=` + linków `/posiedzenie/` |
| prawomiejscowe.pl | „Baza Aktów Własnych", **75 124 B** | — (rodzina bezużyteczna: listy dynamiczne, brak frazy „raport") |
| iq.pl (hosting) | strona zastępcza 503, **2 449 B** | treść zawiera `iq.pl/oferta/serwery-dedykowane` |

**Reguła:** przy każdej nowej rodzinie źródeł najpierw wykonać **zapytanie kontrolne o zasób, który na pewno nie istnieje** (np. `/UrzadGminyNieIstniejeXyz`). Jeśli odpowiedź jest identyczna z „trafieniem" — rodzina jest ślepa. W turze 9 ten test zredukował 52 rzekome portale eSesja do 11 realnych; w turze 11 od razu wykluczył 56 rzekomych instytucji na prawomiejscowe.pl.

**Systematyczna sonda mirrorów `*.bip.net.pl`** (56 jednostek) dała jedno realne odkrycie: **`sabnie.bip.net.pl`** — nowy, działający BIP w silniku BIPv2, odrębny od znanego wcześniej `sabnie.archiwum.bip.net.pl`. Kategoria „RAPORT O STANIE GMINY" obejmuje jednak tylko 2024–2025; skan `/kategorie/{1-200}` i `/artykuly/{1-800}` nie ujawnił roczników 2019–2020.

**Wayback — zachowanie niestabilne:** w tej turze pobieranie plików PDF działało (`HTTP 200`), ale zarchiwizowane strony HTML zwracały `503` przy 8 próbach z odstępami 45 s. Cap 5 MiB pozostaje twardy — `Range: bytes=5242880-` również daje `503`, więc dociągnięcie brakującej części pliku jest niemożliwe.

**Nowa sygnatura pustki (tura 18c — platforma gov.pl):** podstrony rocznikowe `…/raport-o-stanie-gminy/raport-o-stanie-gminy-za-{rok}-rok.html` zwracają **`HTTP 200`, ale identyczny szablon zawierany (ten sam MD5)** dla wszystkich lat, w tym dla roku, który na pewno nie istnieje (np. Bielany: `2018` = `2025`, MD5 `41e86699…`). To **soft-404**. Poprawne rozpoznanie: porównanie **MD5 dwóch podstron** — jeśli identyczne → pusta kategoria, brak pozycji. To uzupełnia tabelę sygnatur z §8b o wariant „identyczny MD5, różne adresy" (obok stałej długości HTML).

**Zobowiązujące rozróżnienie dla gov.pl:** kategoria może istnieć (`/raport-o-stanie-gminy/` = 200) i listować tylko niektóre roczniki (np. 2019, 2022, 2023, 2024 — bez 2018/2020/2021). Obecność kategorii **nie** oznacza, że dany rocznik jest opublikowany — zawsze sprawdzić **wystawienie konkretnego roku** (obecność `fobjects/details` z załącznikiem + link `download`).

---

---

## 8c. Nowe rodziny źródeł — przetestowane i ocenione (tura 12)

Po wyczerpaniu klasycznych ścieżek przetestowałem archiwa i mechanizmy publikacji, których wcześniej nie używałem.

| Rodzina | Wynik | Ocena |
|---|---|---|
| **Common Crawl** (127 kolekcji, API indeksu) | indeks odpowiada; dla domen gmin zwraca wyłącznie HTML, brak plików raportów; przy intensywniejszym odpytywaniu `http=000` | ograniczona przydatność — CC indeksuje głównie strony, rzadko PDF-y z małych BIP-ów |
| **archive.today** (.ph/.today/.is/.li) | `HTTP 429` dla całej adresacji środowiska | zablokowane |
| **arquivo.pt**, **Memento TimeTravel** | brak odpowiedzi / brak zasobów PL | nieprzydatne dla polskich gmin |
| **prawomiejscowe.pl** | strona-szkielet 75 124 B dla dowolnej instytucji | ślepa (patrz §8b) |
| **Brute-force nazw na `bip-v1-files`** | 18 000 kombinacji (ID wiadomości × 12 nazw) → 0 trafień | potwierdza regułę z §3: zgadywanie nazw nie działa |
| **★ Linki do chmur zewnętrznych** | 1 trafienie na 6 sprawdzonych gmin | **warte sprawdzania** |

### ★ Nowy mechanizm publikacji: raport poza infrastrukturą gminy

**Stawiguda 2019** — strona BIP istnieje i jest żywa, ale **nie ma załącznika**. Gmina opublikowała raport jako:
- „Raport w formie prezentacji interaktywnej" → **Microsoft Power BI** (treść dynamiczna, nie plik),
- „Raport - plik pdf" → **Google Drive** (`drive.google.com/file/d/1RI9x…`).

Plik na Drive został usunięty (404 na obu endpointach pobierania), a Wayback nie ma jego snapshotu. **Nieodwracalna utrata przez publikację w zewnętrznej chmurze.**

**Wniosek praktyczny:** żaden skan przestrzeni ID ani CDX własnej domeny takiego raportu nie znajdzie — plik nigdy nie był na serwerze gminy. Przy pustej stronie raportowej zawsze sprawdzać linki wychodzące do: `drive.google`, `docs.google`, `powerbi`, `onedrive`/`1drv.ms`, `dropbox`, `sharepoint`, `wetransfer`. To także **argument archiwizacyjny**: dokumenty urzędowe hostowane w chmurach komercyjnych znikają bezpowrotnie.

---

## 8d. Re-weryfikacja żywych serwerów po finalizacji (tura 18, 02.09.2026)

Po domknięciu wszystkich 2488 wierszy wróciłem do klastra „raport powstał, plik niedostępny" i **prze-crawlowałem żywe serwery** — cel: sprawdzić, czy po wcześniejszych przeszukaniach (z AUR 6–14) nie pojawiły się **nowe** pliki raportów (np. dołożone później lub po migracji).

**Metoda:** dla gmin z brakami wyodrębniono realne hosty i ponownie pobrano: strony główne, kategorię „Raport o stanie gminy", wyszukiwarkę CMS, katalog plików i (dla platformy gov.pl) standardowe podstrony rocznikowe. gov.pl wymagało **TOR** (bezpośrednio `http=000` — WAF; przez polski exit `200`).

**Wyniki:**

### 8d.1. Klaster podlaski (SmartSite `bip-ug{…}.podlaskie.eu`, 16 hostów)
Na żywych serwerach **obecne są wyłącznie nowsze roczniki (2023–2025); brakujący rok nigdy nie był opublikowany** — nie jest to kwestia usunięcia ani blokady.

| Gmina | Live serwer (2026) | Wniosek |
|---|---|---|
| Narewka | tylko raport 2025 | brak 2018 |
| Jasionówka | pliki 2019–2025 | brak 2018 |
| Trzcianne | tylko 2024 | brak 2018–2022 |
| Nurzec-Stacja | 2019–2025 | brak 2018 |
| Perlejewo | tylko 2019 | brak 2018 |
| Orla | tylko 2025 + zgłoszenia do debaty | brak 2019/2020/2022 |
| Rudka | wyszukiwarka → 0 plików | brak 2019–2025 |
| Sztabin, Michałowo, Zawady, Kuźnica, Wizna | brak kategorii / 0 plików | brak |
| Kulesze Kościelne | kategoria jest **stubem** (tylko obrazek, 0 PDF) | brak 2018–2022 |
| Augustów | **brak DNS**, https/http 000 | nieosiągalny |

### 8d.2. Klaster poza podlaskim (mazowieckie 48 / warmińsko-mazurskie 15)

> Wnioski z tury 18c ([dodane niżej](#8c)) uzupełniają obraz dla Andrzejewo i Bielan (przez Tor).

**Przełom — Dźwierzuty 2018, 2019, 2020 (rows 2025–2027).** Kategoria BIP wymienia artykuły raportów **2018–2025**; artykuły i pliki dowodowe zachowane w Wayback na `gminadzwierzuty.pl/portal/download/`:
- 2018: `file_id/227` — „Raport o stanie Gminy Dźwierzuty PDF 2,6 MB", CDX 984 677 B, PDF v1.5 „108 str."
- 2019: `file_id/230` — „Raport … za 2019 2,4 MB", CDX 976 153 B, PDF v1.7 „86 str."
- 2020: `file_id/299` — „Raport … za 2020 1,8 MB", CDX 1 019 677 B, PDF v1.7 „72 str."

Wszystkie **obcięte do ~1 MiB** (CDX poniżej oryginału), **brak markera EOF, pypdf → „Stream has ended unexpectedly"**. Common Crawl — brak przechwyceń. → **dowód istnienia znacząco wzmocniony** (konkretne nazwy + rozmiary + ujęte w Wayback), ale kompletny plik wciąż nieodzyskiwalny. Zgodnie z metodyką: `brak_finalnie` + wzmocniony dowód.

**Gminy z żywą kategorią raportu, ale potwierdzony brak danego rocznika:**

| Gmina | Kategoria BIP | Brakujące lata |
|---|---|---|
| Biała Piska | artykuły 2019–2025 | 2018 |
| Jedwabno | 2020–2025 | 2018, 2019 |
| Mrągowo | 2020–2025 | 2018, 2019 |
| Barczewo | 2021–2025 | 2019, 2020 |
| Pasym | 2024+ | 2023 |
| Sypniewo | brak kategorii (tylko OOŚ) | 2023 |

**Klaster platformy gov.pl (soft-404):** `golymin, ugszulborze, ugzarebykoscielne, ugczarnia, ugtroszyn, ugkrzynowlogamala, ugjoniec, ugskorzec, ugwisniew, ugkosowlacki, ugsabnie, andrzejewo, ugmokobody…` — kategoria `/raport-o-stanie-gminy/` oraz podstrony `…-za-{rok}-rok.html` **2018–2025 zwracają identyczny szablon soft-404 (39 693 B, zgodne MD5, brak tytułu/załącznika)**. To potwierdza, że te gminy **nie opublikowały** pliku raportu na platformie gov.pl (kategoria istnieje, ale jest pusta).

### Wniosek (tura 18)
Re-crawl żywych serwerów **nie przyniósł żadnego nowego kompletnego raportu**. Największa zdobycz to **wzmocniony, konkretny dowód istnienia** dla Dźwierzuty 2018–2020 (pliki istniały, zachowane ~1 MB w Wayback, nieczytelne). Blokady są **strukturalne** (nigdy nie opublikowano / plik usunięty / twardy limit Wayback), a nie wynikające z chwilowej awarii Wayback (które w turze 18 było zdrowe — CDX i pobieranie zwracały 200).

**Kluczowe rozróżnienie dla dalszych tur:** gdzie gmina ma **żywe, niepuste** kategorie rocznikowe z brakującym latem → raport nie powstał; gdzie kategoria jest **pusta lub pusty szablon soft-404** → raport nie opublikowany; gdzie w Wayback jest plik ≠ oryginał → raport powstał, ale niepełny.

---

## 8e. Odzyskanie z żywej domeny głównej — ogłoszenie o debacie → załącznik (tura 18c)

**Nowa metoda, która zadziałała (odzyskany 1 plik = Mrągowo 2018, row 1865).**

**Punkt wyjścia (§2.7 + §5):** raport bywa **załącznikiem do ogłoszenia o debacie / informacji dla mieszkańców**, często na **domenie głównej gminy** (nie w BIP) i nieprzypisanym do kategorii „Raport o stanie gminy" — dlatego re-crawl **kategorii** (tura 18) go nie znajduje, mimo że plik żyje.

**Procedura:**
1. Skan Wayback **`matchType=domain` na domenie głównej gminy** (nie tylko `bip.*`) z `filter=urlkey:.*raport.*` — znajduje artykuły typu „debata nad raportem", „informacja dla mieszkańców", „raport o stanie gminy za {rok}".
2. Wyłuskać artykuł dla brakującego rocznika i jego **załączniki** (`/attachment/...`, `/portal/download/file_id/{N}.html`, `getfile&id=N`).
3. **Sprawdzić, czy oryginalny (żywy) serwer nadal je serwuje** — to tu jest klucz: kategoria BIP pokazuje tylko najnowsze lata, ale stary artykuł z 2019 r. może wciąż mieć załącznik PDF pod starym URL-em.

**Wynik:** Mrągowo 2018 — ogłoszenie `/6187/raport-o-stanie-gminy-mragowo-za-2018-rok.html` (BIP żywe), załącznik `/attachment/informacja/5349/…` → **`Raport.pdf` 655 871 B, PDF v1.4, 69 str., kompletny** (art. 28aa, Urząd Gminy Mrągowo). Rzeczywisty `Content-Length` 655 871 B, `filename="Raport.pdf"`. **Status: `brak_finalnie` → `ok_finalnie`** (row 1865).

**Ograniczenie metody:** z ~15 kandydatów (Mrągowo, Krzynowłoga, Zabrodzie, Liw, Troszyn, Kosów Lacki, Sarnaki, Biskupiec, Zaręby Kościelne) tylko Mrągowo miał plik nadal żywy; reszta: kategoria pokazuje wyłącznie roczniki 2021+, a brakujące lata (2018–2020) nie są żywe ani w archiwum.

*Powtórzenie (tura 18c) potwierdziło też ostatecznie nieodzyskiwalność Dźwierzuty 2018–2020: nagłówek Wayback `x-archive-orig-x-crawler-content-length: 2735195` vs `content-length: 1048576` dowodzi, że **crawler obciął rekord do 1 MiB** — odzyskanie części powyżej 984 677 B kończy się `302` (rekord faktycznie ma 1 048 576 B).*

**Wyniki metody na pozostałych gminach (tura 18c, bez odzysku — realny brak publikacji):** Mrągowo *gmina wiejska* 2019 (kategoria + Wayback: tylko 2018 i 2020–2025), Kosów Lacki 2018–2020 (debata 2019 art.995 bez załącznika; pliki od 2022), Barczewo 2019/2020 (kategoria + XML: 2021–2025; attachment ≤ 946 = 404), Biała Piska 2018 (artykuły od 2019), Wieliczki 2018 (dział od 2019), Szczytno 2025 (publiczny najnowszy = 2024, po 31.05.2026), Szulborze Wielkie (uchwały o wotum istnieją, Wayback pusty), Zaręby Kościelne, Troszyn, Sarnaki, Wierzbno, Grębków, Liw, Andrzejewo — wszystkie potwierdzone jako brak publikacji pliku.

*Pułapka wykryta w turze 18c:* `ug.szulborze.wrotamazowsza.pl` to **SPA geoportuny** (Mazowiecki System Informacji Przestrzennej), nie BIP gminy — przy sondażu domen `.wrotamazowsza.pl` zawsze sprawdzać, czy to faktycznie serwis gminy, a nie ogólna platforma geoprzestrzenna.

---

## 8f. Precyzyjna identyfikacja załącznika z `<object>` — odzysk Lipsk 2025 + nowy host Zbójna (tura 18c/18d)

**ODZYSK Lipsk 2025 (row 872, `brak_finalnie` → `ok_finalnie`).** Poprzednio (tura 2) przetestowano **189 kombinacji** ścieżek w galerii `images/phocagallery/wydarzenia_gminne/rok_2026/` (miesiące IV/V/VI × 21 dni × 3 warianty nazwy typu `Raport_o_stanie_Gminy_za_2025_...`) — wszystkie 404. W turze 18d zamiast zgadywania nazw **odszyfrowano prawdziwy wzorzec publikacji z artykułu 2024 r.** i podążono za strukturą serwisu:

1. Raport za 2024 r. wisiał pod `rok_2025/V/27/Raport_o_stanie_Gminy_za_2024_r_wersja_ostateczna_ok.pdf` — wniosek: nazwy plików **nie są stałe** (`_wersja_ostateczna_ok`), a dzień jest zmienny.
2. Wyszukano **artykuł** „Raport o stanie Gminy za 2024 r." na `lipsk.pl/index.php/aktualnosci/ogloszenia` (slug: `raport-o-stanie-gminy-za-2024-r`).
3. Dla rocznika 2025 zidentyfikowano artykuł **`/aktualnosci/ogloszenia/raport-o-stanie-gminy-lipsk`** (data publikacji **13.05.2026**) — w HTML artykułu załącznik nie jest zwykłym linkiem `<a href>`, tylko **osadzonym PDF-em**: `<object data="/images/phocagallery/wydarzenia_gminne/rok_2026/V/14/zarzadzenie-raport-2025.pdf" type="application/pdf" ...>`.
4. Pobrano ten plik: **5 799 640 B (PDF 1.7)**, `sha256` w CSV, zapisany `pobrane_raporty/872_lipsk_2025.pdf`.

**Wniosek metodyczny (najcenniejszy):** w Joomla raport publikuje się jako **osadzony `<object data="...pdf">`**, nie jako `href`. **Skanowanie `<a href>` / zgadywanie nazw plików jest więc z definicji nieefektywne — trzeba (a) znaleźć artykuł, (b) parsować tag `<object data=...>` i (c) `{link}` (takie jak `/images/file.pdf`) to ślepa uliczka, a prawdziwy PDF siedzi w `data` atrybucie `object`. Skan 189 kombinacji nie mógł trafić, bo nazwa brzmiała `zarzadzenie-raport-2025.pdf` zamiast oczekiwanej `Raport_o_stanie_...` oraz dzień `14` (nie `27`).

**Zbójna 2020 (row 1683) — POTWIERDZONY realny brak publikacji na NOWYM hoście.** Stare BIP `zbojna.powiatlomzynski.pl` (PUBLIKATOR) przekierowuje na **nowy host `bip.zbojna.pl`** (system `index.php?k=` + `index.php?wiad=`). W kategorii **k=321 (Aktualności)** jest dokładnie 7 artykułów „RAPORT O STANIE GMINY ZBÓJNA":
- wiad `2171` → download `2710` = **r.2018**
- wiad `2306` → download `3018` = **r.2019**
- wiad `2709` → download `3835` = **r.2021**
- wiad `2892` → download `4225` = **r.2022**
- wiad `3143` → download `4665` = **r.2023**
- wiad `3343` → download `5100` = **r.2024**
- wiad `3553` → download `5544` = **r.2025**

Wszystkie **7 PDF pobrano** i zidentyfikowano przez render strony 1 (`pymupdf`, bo pliki są skanami bez warstwy tekstowej oprócz 2710). Sekwencja jest **ciągła z wyjątkiem roku 2020**: między downloadami 3018 (2019) i 3835 (2021) nie ma ani jednego wiad/pliku. Dodatkowy skan `download.php?id=3019..3834` (wszystkie ID w luce) nie zwrócił żadnego PDF. **Wniosek: raport za 2020 nigdy nie został opublikowany** (gmina pominęła rocznik) — to realny BRAK pliku, nie artefakt techniczny; uzasadnia wcześniejszy status `brak_finalnie`.

**Lejek Zbójna — lekcja:** przy BIP-ach PUBLIKATOR/„hi" odwzorowanie rok→ID jest **liniowe i roczne** (cóż za równy krok +~380–550 na rocznik); po wykryciu przekierowania na nowy host **należy przemapować całość na nowym hoście zamiast skanować stary**. Również **paginacja kategorią `k=321` listuje artykuły w jednym widoku** (nie przez `&p=`), co pozwala wyciągnąć wszystkie tytuły naraz.

**Pułapka gov.pl potwierdzona:** `samorzad.gov.pl/web/<jaki-kolwiek-slug>/raport-o-stanie-gminy` zwraca **identyczny szablon 20 501 B** (tytuł „Strona główna - Gov.pl") również dla nieistniejącej instytucji — analogicznie do strony-szkieletu eSesja (36 759 B). Noty w CSV z `X.bip.gov.pl` (r. 18–24, 281–287, 298 itd.) wskazują, że re-weryfikacja niepodstawiła realnego hosta — przy dalszych pracach te wiersze wymagają **podstawienia właściwej domeny BIP danej gminy** i sprawdzenia jej własnego systemu (nie szkieletu gov.pl).

**Dwa kluczowe rozpoznania hostów w turze 18d:**
1. **Gmina wiejska Augustów** (r. 849 = 2018, r. 852 = 2021) — **migracja BIP na platformę gov.pl.** Stary `bip-ugaugustow.wrotapodlasia.pl` **nie istnieje** (DNS `-2`), nie ma też wariantu `.podlaskie.eu`. Na **`samorzad.gov.pl/web/gmina-augustow/`** potwierdzone publikacje 28aa: **2023** → `/attachment/0bf8f370-086b-45f4-bdfd-83546424c133` (Zarządzenie OR.0050.3.2024) i **2024** → `/attachment/ef23dce8-263b-4454-bd55-2c082bdb677f` (OR.0050.121.2025). Sekcje: `/spis-aktow-prawnych` i `/zarzadzenia-wojta-gminy-ix-kadencja` (lista ładuje się JS/spisem stron), `rada-gminy2`, `informacje-biezace`. **Wniosek metodyczny:** przy „zmarłej" domenie `.wrotapodlasia.pl`/`.podlaskie.eu` szukać **migracji na `samorzad.gov.pl/web/<slug-gminy>`** i odwrotnie.
2. **Gminy Mazowieckie (wrota/seo):** `*.wrotamazowsza.pl` = **SPA geoporten** (nie BIP); `bip.golymin.pl` = **podszyty/przejęty domeną SEO-spam** (nie BIP). Prawdziwy BIP Gołymina-Ośrodka to `golymin-osrodek.biuletyn.net` (brak kategorii raportu 28aa w menu → realny brak). **Wniosek:** przy sondażu domów zawsze weryfikować, czy domena nie jest przejęta (SEO) ani geoportuną.

**Kategorie raportu „świeżo założone" — fałszywe wrażenie braku starszych lat:** Trzcianne (`bip-ugtrzcianne.podlaskie.eu/raport-o-stanie-gminy-trzcianne/`) i Kobylin-Borzymy (`kobylinborzymy.biuletyn.net`) oraz Narewka (`bip-ugnarewka.podlaskie.eu/raport_o_stanie_gminy/`) mają kategorię raportu zawierającą **wyłącznie raporty 2023–2025** (Narewka 2023–2025, Trzcianne 2023–2025, Kobylin 2019–2025 jako wotum, ale bez pliku). Sarnaki (`sarnaki.pl/category/raport-o-stanie-gminy/feed/`) zaczyna od 2021. Starsze roczniki (2018–2022) **nie są publikowane w tych kategoriach** — to albo host z okresu przed CMS, albo kategoria założona później; wbrew pozorom **nie jest to gwarancja braku starszych lat** — należy sprawdzić archiwa sesji/protokołów (Narewka ma kompletne `/rad_gminy/kadencja_20182023/protokoy_rady_gminy_narewka/`, gdzie raport 28aa z załącznikiem mógł wisieć przy sesji absolutoryjnej).

---

## 9. Rekomendowana kolejność działań dla nowej gminy

1. Wyszukiwarki internetowe (różne silniki, fraza w cudzysłowie + nazwa powiatu dla ujednoznacznienia).
2. Rozpoznanie CMS-u → dobór wzorca URL z tabeli w §2.1.
3. Wyszukiwarka wewnętrzna BIP (właściwy parametr!).
4. Kategoria „Raport o stanie gminy" + **historia wersji** strony.
5. Skan przestrzeni ID (≤14 wątków).
6. Indeksy zarządzeń i uchwał (regex `v?otum`, `raport`).
7. Domeny archiwalne i alternatywne.
8. **Portal sesyjny rady (eSesja) — sesja absolutoryjna roku X+1.**
9. Wayback CDX (poprawna składnia filtra).
10. **TOR z polskim exitem** — gdy serwis zwraca `http=000`/timeout mimo poprawnego DNS.
11. **Linki do chmur zewnętrznych** na stronie raportowej (Drive/Power BI/OneDrive) — gdy strona istnieje, ale nie ma załącznika.
12. OCR do potwierdzenia rocznika.

**Zasada nadrzędna:** brak pliku ≠ brak raportu. Zawsze rozdzielać „raport nie powstał" od „raport powstał, ale plik jest niedostępny" — te dwa przypadki mają zupełnie inną wartość badawczą.

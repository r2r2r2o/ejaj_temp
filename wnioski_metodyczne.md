# Wnioski metodyczne — pozyskiwanie raportów o stanie gminy (art. 28aa u.s.g.)

Stan na 01.09.2026. Zbiór: 2488 wierszy (264 gminy × 8 roczników 2018–2025 + 47 jednostek nadrzędnych).
Wynik: **2359 `ok_finalnie` / 129 `brak_finalnie` / 0 `BRAK`** — 94,8% pokrycia.

---

## 1. Skuteczność źródeł (2359 zdobytych plików)

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

## 6. Struktura 129 pozostałych braków

| Liczba | Kategoria |
|---|---|
| 73 | Brak publikacji / brak jakiegokolwiek śladu — gmina nigdy nie opublikowała pliku |
| 48 | **Raport powstał (dowód: uchwała o wotum / ogłoszenie o debacie), plik niedostępny** |
| 7 | Jednostka nie istniała (Grabówka — utworzona 1.01.2025) |
| 1 | Rocznik jeszcze nieopublikowany (Szczytno 2025) |

Dominujący wzorzec: **BIP-y zaczynają publikować raporty dopiero od 2020–2023**, mimo że obowiązek istnieje od 2019 (za 2018). Rocznik 2018 to najsłabiej udokumentowany rok w całym zbiorze.

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

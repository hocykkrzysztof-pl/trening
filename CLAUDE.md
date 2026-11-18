# Trening w domu - notatka projektu

## Cel
Prosta aplikacja webowa do śledzenia treningów w domu: trening siłowy z masą ciała (przysiady, pompki, podciągnięcia, deska) oraz marsz na bieżni. Cel użytkownika: redukcja sylwetki "skinny fat" przez progresję ćwiczeń, marsz i deficyt kalorii.

Użytkownik korzysta głównie z telefonu. Apka ma być hostowana na GitHub Pages.

## Stan obecny
- `index.html` (HTML + CSS + JS inline, bez frameworków i bez builda)
- PWA: `manifest.webmanifest`, `sw.js` (service worker), ikony w `icons/`. Apka instaluje się na telefonie i działa offline. Po zmianie plików podbić `VERSION` w `sw.js`
- Wszystkie ścieżki względne (GitHub Pages serwuje apkę pod `/<nazwa-repo>/`)
- Dane w `localStorage`, klucz `trening-state-v2`; niezapisany trening (draft) w kluczu `trening-draft-v1`, przetrwa przeładowanie i edycję planu
- Interfejs po polsku
- Fonty: Barlow i Barlow Condensed z Google Fonts
- Jasny i ciemny motyw (`prefers-color-scheme`, kolory jako zmienne CSS na `:root`)

## Funkcje
**Zakładka Trening**
- Nagłówek z datą i informacją, co jest dziś w planie (wg harmonogramu aktywnego planu)
- Licznik treningów w bieżącym tygodniu (siłowe i marsze vs cel z harmonogramu)
- Przełącznik: Trening siłowy / Marsz
- Siłowy: dla każdego ćwiczenia wybór wersji (poziomu), pola na wynik każdej serii, przycisk "Pomiń dziś"
- "Cel na dziś": wyliczany z ostatniego treningu w tej samej wersji (wersja rozpoznawana po nazwie, więc zmiana kolejności wersji w edytorze nie psuje celu; zmiana nazwy wersji resetuje cel do min), +1 powtórzenie lub +5 s na serię, w granicach zakresu min-max. Przy nowej wersji cel = min
- Gdy wszystkie serie w ostatnim treningu osiągnęły max, apka proponuje przejście na kolejną wersję
- Ćwiczenia na czas (jednostka `s`): przycisk Start pod każdą serią, 3 s odliczania, liczenie w górę, wibracja i sygnał dźwiękowy po osiągnięciu celu, Stop wpisuje czas do pola
- Pływający timer przerwy 90 s (liczy od czasu końca, więc nie spóźnia się w tle; dźwięk przez jeden współdzielony AudioContext odblokowywany przy dotknięciu)
- Marsz: data, czas (min), tempo (km/h), nachylenie (%)

**Zakładka Historia**
- Wykresy słupkowe (inline SVG) sumy powtórzeń z ostatnich 10 treningów dla każdego ćwiczenia aktywnego planu oraz czasu marszu
- Lista wszystkich sesji, usuwanie przez podwójne kliknięcie (bez `confirm()`)

**Zakładka Pomiary**
- Waga (kg) i obwód w pasie (cm), zmiana od pierwszego pomiaru, wykres obwodu pasa

**Zakładka Plany**
- Lista planów: ustaw jako aktywny, edytuj, duplikuj, usuń
- Edytor planu: nazwa, harmonogram tygodnia (Siłowy / Marsz / Wolne dla każdego dnia), ćwiczenia z liczbą serii i listą wersji (nazwa, jednostka pow./sek., zakres od-do), kolejność ćwiczeń
- Kopia danych: eksport do pliku JSON, import z pliku lub wklejonego tekstu (import zastępuje wszystkie dane)

## Model danych
```js
state = {
  v: 2,
  activePlan: 'plan-dom',
  plans: [{
    id, name,
    days: { 0..6: 'strength' | 'walk' | 'rest' },   // 0 = niedziela
    exercises: [{ key, name, sets, variants: [{ n, u: 'pow'|'s', min, max }] }]
  }],
  levels: { [planId]: { [exerciseKey]: indexWersji } },
  sessions: [
    { id, type: 'strength', planId, date: 'YYYY-MM-DD',
      ex: { [exerciseKey]: { name, n, u, variant, sets: [liczba | ''], skip } } },
    { id, type: 'walk', date, min, speed, incline }
  ],
  measures: [{ id, date, weight, waist }]
}
```
Sesje siłowe zapisują nazwę ćwiczenia, nazwę wersji i jednostkę, żeby historia była czytelna po edycji lub usunięciu planu.

**Format v1** (eksport ze starej wersji apki na claude.ai): `{ v:1, variants:{squat,push,pull,plank}, sessions, measures }`, bez `planId` i bez nazw w sesjach. Funkcja `normalize()` konwertuje go do v2 i przypisuje do planu domyślnego. Tę kompatybilność trzeba zachować.

## Znane ograniczenia
- Dane tylko w jednej przeglądarce, brak synchronizacji między urządzeniami
- Wszystko w jednym pliku
- Timer przerwy ma stałe 90 s
- Brak testów w repo (testy przeglądarkowe przez headless Chrome + CDP były robione jednorazowo)

## Propozycje do rozważenia (do decyzji użytkownika)
- Synchronizacja między urządzeniami (np. Supabase, Firebase albo plik w GitHub Gist)
- Konfigurowalny czas przerwy, osobno dla każdego ćwiczenia
- Przypomnienie o eksporcie kopii, np. gdy ostatnia kopia jest starsza niż 2 tygodnie
- Rozbicie na osobne pliki (HTML, CSS, JS)
- Edycja zapisanej sesji, nie tylko usuwanie

## Zasady
- Interfejs po polsku
- W tekstach nie używać długich myślników, tylko zwykły łącznik (-)
- Zachować kompatybilność z eksportem v1 i v2
- Nie używać `alert()`, `confirm()`, `prompt()`, potwierdzenia przez podwójne kliknięcie
- Mobile first, z obsługą safe area (`env(safe-area-inset-*)`)
- Po zmianach przetestować: zapis treningu, cel na dziś, propozycję kolejnej wersji, timer deski, edytor planu, eksport i import (v1 i v2), usuwanie wpisów

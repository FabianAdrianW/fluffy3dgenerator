# Fluff Generator — prototyp koncepcyjny

Interaktywny prototyp doświadczenia zakupowego dla marki kosmetyków naturalnych **Fluff** ([fluff.com.pl](https://fluff.com.pl)): użytkownik składa własnego stworka ("Fluffa") w 3D, a ten zostaje z nim na stronie i pomaga dobrać kosmetyki według nastroju.

---

## Czym to jest, a czym nie jest

To **koncept powstały jako zadanie rekrutacyjne**, nie wdrożenie produkcyjne i nie oficjalna strona marki. Marka Fluff nie brała udziału w jego powstaniu; nazwa, logo, zdjęcia i teksty produktowe należą do Fluff i użyte są wyłącznie poglądowo.

Prototyp **powstał metodą spec-and-verify z użyciem AI**: pisałem specyfikację, diagnozowałem błędy z zachowania interfejsu, wyznaczałem kierunek i weryfikowałem efekt; kod pisany był w dużej mierze przez asystenta AI pod moim nadzorem. Piszę to wprost, bo repo ma pokazywać metodę pracy, a nie sugerować autorstwo silnika graficznego od zera.

| Element | Status |
|---|---|
| Generator 3D, interakcje, UI | działa w całości, po stronie przeglądarki |
| Katalog produktów | wycinek oferty wpisany ręcznie; docelowo z Shopera |
| Mapowanie nastrój → produkt | zaszyte w JS; docelowo tagi w backendzie |
| Karta produktu (PDP) | brak — linki prowadzą do prawdziwych kart na fluff.com.pl |
| Czat | lokalna baza wiedzy; tryb LLM wyłączony (patrz niżej) |
| Zakup, koszyk, płatności | brak |

## Stack

Vanilla HTML/CSS/JS, bez frameworka i bez kroku budowania. Trzy pliki to całość aplikacji — otwierasz `index.html` i działa.

- **Three.js r128** (CDN) — baza pod własny renderer futra
- **Shell rendering** — futro jako stos warstw powłoki z szumem i grawitacją, liczony per gatunek
- **Poppins** (Google Fonts), obrazy produktowe z CDN Fluffa

## Co było najtrudniejsze

Pięć rzeczy, które wymagały diagnozy zamiast doklejenia biblioteki:

1. **Uszy jako osobne meshe.** Przy uszach doklejonych do bryły ciała zwiększanie ich rozmiaru rozciągało czoło. Rozdzielenie geometrii rozwiązało problem u źródła.
2. **Odwrócone normalne** po przebudowie geometrii — futro rosło do wnętrza bryły.
3. **Kierunek stycznej (`sdir`).** Bez rzutowania stycznej na płaszczyznę prostopadłą do normalnej futro wyglądało na spikselowane przy odchyleniu.
4. **Atrybut per-wierzchołek `combW`** — bramkuje efekt czesania wyłącznie do ciała, żeby głaskanie nie przenosiło się na uszy.
5. **`furAt()` plus wyszukiwanie binarne** przy osadzaniu akcesoriów. Stała wartość uniesienia zawodzi, bo wysokość futra zmienia się drastycznie punkt po punkcie (maseczka na twarz zbija je nawet o 80%) — trzeba odtworzyć tę samą formułę, którą buduje się futro.

Test szczelności: 464 kombinacje gatunek × akcesorium, 91,9% wypełnienia koła, zero wyjść poza jego obrys.

## Czat i bezpieczeństwo

Czat domyślnie działa na **lokalnej bazie wiedzy** — bez żadnego wywołania sieciowego. Zanim cokolwiek trafi do wyszukiwania, tekst przechodzi przez warstwę bramek: kryzys → wstrzyknięcie promptu → pytania medyczne → dane osobowe → poza zakresem → brak wiedzy. Bramka kryzysowa istnieje dlatego, że cały produkt pyta „jaki masz nastrój?" — jeśli zapraszasz ludzi do mówienia o samopoczuciu, część powie rzeczy, na które sklep nie jest odpowiedzią.

Architektura odwołuje się do OWASP Top 10 for LLM Applications oraz obowiązku ujawnienia z art. 50 AI Act.

**O trybie LLM:** w kodzie jest opcjonalna ścieżka do modelu (`window.FLUFF_CHAT`), domyślnie wskazująca na `api.anthropic.com`. **Z przeglądarki ona nie zadziała i tak ma być** — klucz API w przeglądarce to klucz oddany każdemu, kto naciśnie F12. Docelowo `endpoint` ma wskazywać własny serwer-pośrednik, który dokłada klucz, trzyma limity na IP i loguje rozmowy do przeglądu. W tym repo nie ma żadnego klucza i nigdy nie było.

## Uruchomienie

```bash
git clone <adres-repo>
cd <repo>
python3 -m http.server 8000
# http://localhost:8000
```

Otwarcie `index.html` bezpośrednio z dysku też działa (potrzebny internet na three.js i zdjęcia). Na iOS trzeba użyć przeglądarki — podgląd plików nie uruchamia WebGL.

## Pliki

```
index.html      generator 3D + czat
produkty.html   lista produktów z filtrem nastroju
favicon.svg
.nojekyll       wyłącza przetwarzanie Jekyllem na GitHub Pages
```

Silnik jest wklejony inline w obu plikach — tak samo w każdym. To świadomy kompromis prototypu: jeden plik = jedna strona, którą można wysłać, otworzyć i pokazać bez środowiska.

## Licencja

Kod: MIT (plik `LICENSE`). Nazwa, logo, zdjęcia i teksty produktowe Fluff pozostają własnością marki i nie są objęte tą licencją.

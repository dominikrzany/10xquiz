# Dokument wymagań produktu (PRD) - 10xQuiz

## 1. Przegląd produktu

10xQuiz to interaktywna platforma do tworzenia i rozgrywania quizów, która łączy naukę z grywalizacją i rozrywką. Aplikacja wykorzystuje sztuczną inteligencję (GPT-4) do automatycznego generowania pytań quizowych z materiałów dostarczonych przez użytkownika.

Kluczowe cechy produktu:
- Automatyczne generowanie pytań z PDF, zdjęć, linków lub tekstu za pomocą AI
- System PIN-ów umożliwiający łatwe udostępnianie quizów
- Asynchroniczna rozgrywka - każdy gracz rozwiązuje quiz we własnym tempie
- Globalna tablica wyników (leaderboard) dla każdego quizu
- System punktacji zachęcający do szybkich i poprawnych odpowiedzi (maksymalnie 1000 punktów)
- Bez wymogu rejestracji dla graczy - tylko podanie pseudonimu

Stos technologiczny:
- Frontend: Astro 5, React 19, TypeScript 5, Tailwind CSS 4, Shadcn/ui
- Backend: Supabase (PostgreSQL + Authentication + API)
- AI: OpenRouter.ai (dostęp do GPT-4 i innych modeli)
- Hosting: Vercel/Netlify (frontend) + Supabase Cloud (backend)

Zakres MVP:
Minimum Viable Product skupia się na podstawowych funkcjonalnościach umożliwiających tworzenie quizów za pomocą AI, rozgrywanie ich indywidualnie i konkurowanie z innymi na tablicy wyników. MVP nie obejmuje zaawansowanych analiz, płatnych funkcji, trybu real-time ani integracji zewnętrznych.

## 2. Problem użytkownika

Główny problem:
Tradycyjna nauka i testowanie wiedzy są monotonne i mało angażujące. Istniejące rozwiązania mają następujące wady:
- Ręczne tworzenie quizów jest czasochłonne (nauczyciele spędzają godziny na przygotowywaniu materiałów)
- Brak narzędzi do szybkiej konwersji materiałów edukacyjnych na interaktywne quizy
- Dostępne platformy są skomplikowane lub wymagają płatnych subskrypcji
- Tradycyjne formy testowania wiedzy nie motywują uczniów do nauki
- Brak elementu rywalizacji który motywuje do lepszych wyników

Segmenty użytkowników:
1. Nauczyciele i trenerzy - potrzebują narzędzia do szybkiego tworzenia quizów z materiałów lekcyjnych
2. Studenci - chcą testować swoją wiedzę w konkurencyjny sposób
3. Grupy społeczne (przyjaciele, rodzina) - szukają angażującej rozrywki
4. Każdy, kto potrzebuje stworzyć quiz i zmotywować innych do nauki

Bolączki użytkowników:
- Brak czasu na ręczne tworzenie zestawów pytań
- Wysokie koszty istniejących rozwiązań edukacyjnych
- Skomplikowane interfejsy i krzywa uczenia
- Nudne formy testowania wiedzy dla uczących się
- Trudności w motywowaniu do nauki i testowania wiedzy

Rozwiązanie:
10xQuiz automatyzuje proces tworzenia quizów przy użyciu AI, eliminując czasochłonną pracę manualną, i oferuje angażującą rozgrywkę z elementami grywalizacji (punkty, leaderboard). Dzięki prostemu udostępnianiu przez PIN i asynchronicznej rozgrywce, każdy może grać we własnym tempie, a element konkurencji motywuje do lepszych wyników.

## 3. Wymagania funkcjonalne

### 3.1 Autentykacja i zarządzanie użytkownikami

3.1.1 Rejestracja użytkownika (twórcy quizów)
- Formularz rejestracji z polami: email, hasło (min. 8 znaków), pseudonim
- Walidacja formatu email i unikalności w systemie
- Hashowanie hasła przez Supabase Auth (automatyczne)
- Automatyczne logowanie po rejestracji
- Przekierowanie do dashboard po sukcesie

3.1.2 Logowanie użytkownika
- Formularz logowania z polami: email, hasło
- Sesja przechowywana w Supabase Auth (JWT token)
- Przekierowanie do dashboard po sukcesie
- Obsługa błędów: "Nieprawidłowy email lub hasło"

3.1.3 Middleware autoryzacji
- Protected routes wymagające zalogowania: /dashboard, /quiz/new, /quiz/:id/edit
- Automatyczne przekierowanie do /login dla niezalogowanych użytkowników
- Weryfikacja JWT token przy każdym request do API

3.1.4 Gracze (bez rejestracji)
- Gracze NIE muszą się rejestrować
- Przy rozpoczęciu quizu podają tylko pseudonim
- Pseudonim zapisywany w sesji gry (game_sessions tabela)

### 3.2 Tworzenie i zarządzanie quizami

3.2.1 Dashboard użytkownika
- Lista wszystkich quizów użytkownika z następującymi informacjami:
  - Tytuł quizu
  - Liczba pytań
  - 6-cyfrowy PIN
  - Data utworzenia
  - Liczba rozegranych gier
  - Przycisk "Kopiuj PIN" (clipboard)
- Sortowanie: najnowsze na górze (created_at DESC)
- Akcje dla każdego quizu: "Edytuj", "Zobacz wyniki", "Usuń"
- Przycisk CTA: "Stwórz nowy quiz"
- Empty state: "Nie masz jeszcze żadnych quizów. Stwórz pierwszy!"

3.2.2 Tworzenie quizu - formularz
Struktura formularza:
- Pole: Nazwa quizu (required, max 100 znaków)
- Sekcja AI Generator (opcjonalna):
  - Textarea dla instrukcji tekstowych: "Opisz jaki quiz chcesz stworzyć (poziom trudności, liczba pytań, temat)"
  - Input type=file dla uploadu (PDF max 10MB, zdjęcia max 5MB)
  - Przycisk "Generuj pytania" z loading state
- Dynamiczna lista pytań:
  - Każde pytanie: textarea (max 500 znaków)
  - 4 odpowiedzi (A, B, C, D) z checkboxami do zaznaczenia prawidłowych
  - Każda odpowiedź: input tekstowy (max 200 znaków)
  - Przycisk "Dodaj pytanie" / "Usuń pytanie"
- Przycisk "Zapisz quiz i wygeneruj PIN"

Walidacja:
- Minimalna liczba pytań: 3
- Maksymalna liczba pytań: 20
- Każde pytanie musi mieć dokładnie 4 odpowiedzi
- Co najmniej 1 odpowiedź musi być zaznaczona jako prawidłowa
- Wszystkie pola są wymagane

3.2.3 AI generowanie pytań
- Akceptowane formaty: PDF, JPG, PNG, plain text, link do artykułu
- Timeout: maksymalnie 60 sekund
- OpenRouter.ai API (GPT-4 lub inny model) analizuje materiał
- Po wygenerowaniu: automatyczne wypełnienie formularza
- Użytkownik może edytować wygenerowane pytania przed zapisaniem
- Pliki NIE są zapisywane w systemie (tylko do analizy)
- Rate limiting: 5 generacji na użytkownika na dzień (kontrola kosztów)

Obsługa błędów:
- Timeout (>60s): "Generowanie trwa zbyt długo, spróbuj ponownie"
- Błąd API: "Nie udało się wygenerować pytań. Spróbuj dodać więcej treści lub zmień materiał."
- Zbyt mało tekstu: "Materiał jest zbyt krótki. Dodaj więcej treści (minimum 100 znaków)."
- Plik za duży: "Plik jest zbyt duży. Maksymalny rozmiar: 10 MB (PDF) / 5 MB (zdjęcia)"

3.2.4 Automatyczne generowanie PIN
- Po zapisaniu quizu system automatycznie generuje 6-cyfrowy PIN (000000-999999)
- Sprawdzenie unikalności PIN w bazie danych (kolumna quizzes.pin UNIQUE)
- Jeśli collision: retry z nowym PIN (do 3 prób)
- PIN jest stały dla danego quizu (nie zmienia się)
- PIN wyświetlany w dashboardzie przy quizie
- Możliwość kopiowania PIN do schowka jednym kliknięciem

3.2.5 Edycja quizu
- Użytkownik może edytować tylko swoje quizy (weryfikacja user_id)
- Identyczna struktura formularza jak przy tworzeniu
- Brak sekcji AI Generator (tylko edycja manualna)
- Zmiany w quizie NIE wpływają na już rozegrane sesje
- PIN pozostaje ten sam po edycji
- Przycisk "Zapisz zmiany"

3.2.6 Usuwanie quizu
- Modal konfirmacji: "Czy na pewno chcesz usunąć quiz '{title}'? Zostaną usunięte również wszystkie wyniki gier."
- Przyciski: "Anuluj" / "Usuń"
- Cascade delete: usunięcie quizu usuwa powiązane game_sessions (wyniki)
- Toast notification: "Quiz został usunięty"

3.2.7 Podgląd wyników quizu przez HOST-a (Analytics)
**Route:** `/quiz/:id/results` (wymaga auth + ownership)

**Cel:** HOST może monitorować jak gracze radzą sobie z jego quizem

**Fetch danych:**
- GET /api/quizzes/:id/analytics
- Response: 
  ```json
  {
    quiz_title: string,
    total_games: number,
    completed_games: number,
    abandoned_games: number,
    average_score: number,
    leaderboard: [{
      rank: number,
      player_nickname: string,
      total_points: number,
      correct_answers: number,
      total_questions: number,
      completed_at: string
    }]
  }
  ```

**Wyświetlenie - sekcja statystyk:**
- Tytuł quizu (header)
- Karty ze statystykami:
  - "Rozegranych gier": {total_games}
  - "Ukończonych": {completed_games} ({percentage}%)
  - "Porzuconych": {abandoned_games}
  - "Średni wynik": {average_score} pkt
  - "Średni % poprawnych": {average_percentage}%

**Wyświetlenie - leaderboard (wszystkie wyniki):**
- Dropdown filtrowania:
  - "Wszystkie czasy" (default)
  - "Ostatnie 24h"
  - "Ostatni tydzień"
- Tabela wszystkich graczy:
  - Kolumny: Pozycja | Pseudonim | Punkty | Poprawne odpowiedzi | % | Kiedy
  - Sortowanie: total_points DESC, completed_at ASC
  - TOP 3 z ikonami podium: 🥇 🥈 🥉
  - Paginacja: 20 wyników na stronę
- Search: możliwość wyszukania gracza po pseudonimie

**Funkcjonalności:**
- Export do CSV (przyszłość, nie MVP)
- Przycisk "Odśwież" do reload danych
- Link do quizu: "Udostępnij PIN: {pin}" z przyciskiem "Kopiuj"

**Przyciski akcji:**
- "Edytuj quiz" → `/quiz/:id/edit`
- "Wróć do dashboardu" → `/dashboard`

**Filtrowanie:**
- GET /api/quizzes/:id/analytics?filter=24h|7d
- Backend: WHERE completed_at >= NOW() - INTERVAL '...'

**Empty state:**
- Jeśli brak gier: "Nikt jeszcze nie rozwiązał tego quizu. Udostępnij PIN: {pin}"

### 3.3 Rozgrywka quizu (Route: /play/:pin)

Cała rozgrywka odbywa się w jednym route `/play/:pin` z dynamicznym renderowaniem widoków na podstawie React state. 

**Architektura:**
- Jeden React component: `<QuizPlay />`
- State management: `gameState` określa aktualny widok
- Brak przełączania między route'ami
- Płynne przejścia między etapami gry

**Flow stanów:**
```
nickname_input → countdown → question (reading → answering → feedback) → results
```

**API Flow:**
- Jeden uniwersalny endpoint: `POST /api/game/:sessionId`
- Różne akcje oparte na polu `action` w request body:
  - `{action: 'get_question', question_index: number}` → zwraca pytanie
  - `{action: 'submit_answer', question_id, answer_index, time_taken_ms}` → zapisuje odpowiedź, zwraca feedback + flagę is_last_question
  - `{action: 'get_results'}` → zwraca wyniki końcowe z TOP 10
- Backend zarządza stanem sesji, oblicza punkty, sprawdza czy to ostatnie pytanie
- Uproszczony flow komunikacji: mniej endpointów = łatwiejszy maintenance

---

3.3.1 Wejście na stronę i formularz pseudonimu
**State:** `gameState = 'nickname_input'`

**Flow:**
1. Użytkownik wpisuje PIN na stronie głównej i klika "Rozpocznij quiz"
2. Przekierowanie do `/play/:pin`
3. Component mount: useEffect pobiera dane quizu
   - GET /api/quiz/:pin
   - Response: `{quiz_id, title, total_questions}`
4. Wyświetlenie formularza:
   - Tytuł quizu
   - "{total_questions} pytań"
   - Input dla pseudonimu (3-30 znaków, required)
   - Przycisk "Start"

**Walidacja:**
- PIN musi być 6-cyfrowy (sprawdzone przed przekierowaniem)
- Quiz o danym PIN musi istnieć
- Pseudonim: 3-30 znaków, required

**Akcja po kliknięciu "Start":**
- POST /api/game/start `{quiz_id, player_nickname}`
- Backend tworzy game_session:
  - quiz_id, player_nickname
  - status: 'in_progress'
  - started_at: NOW()
- Response: `{session_id, quiz_title, total_questions}`
- Zapisanie `sessionId` w React state
- Zmiana state: `setGameState('countdown')`

**Błędy:**
- "Quiz o podanym PIN nie istnieje" (404)
- "Pseudonim jest wymagany (3-30 znaków)"

---

3.3.2 Countdown (10 sekund)
**State:** `gameState = 'countdown'`

**Wyświetlenie:**
- Tytuł quizu
- "{total_questions} pytań"
- "Zaczynamy za... 10, 9, 8, 7, 6, 5, 4, 3, 2, 1, START!"
- Duża czcionka, animacja liczb

**Implementacja:**
- Countdown client-side (useState + useEffect z setInterval)
- Po 10 sekundach:
  - `setGameState('question')`
  - `setCurrentQuestionIndex(1)`
  - Fetch pierwszego pytania: POST /api/game/:sessionId `{action: 'get_question', question_index: 1}`

---

3.3.3 Pytanie - faza czytania (5 sekund)
**State:** `gameState = 'question'`, `questionSubState = 'reading'`

**Wyświetlenie:**
- Numer pytania: "Pytanie {currentQuestionIndex}/{total_questions}"
- Treść pytania (duża czcionka, wyśrodkowana)
- Komunikat: "Odpowiedzi pojawią się za..."
- Timer: countdown 5, 4, 3, 2, 1
- Przyciski odpowiedzi UKRYTE (conditional rendering)

**Fetch pytania:**
- POST /api/game/:sessionId `{action: 'get_question', question_index: currentQuestionIndex}`
- Response: `{question_id, question_text, answers: [{text, order_index}], question_number, total_questions}`
- Zapisanie `questionData` w state

**Po 5 sekundach:**
- `setQuestionSubState('answering')`
- Pokazanie 4 przycisków odpowiedzi
- Start 20-sekundowego timera

---

3.3.4 Pytanie - faza odpowiadania (20 sekund)
**State:** `gameState = 'question'`, `questionSubState = 'answering'`

**Wyświetlenie:**
- Numer pytania: "Pytanie {currentQuestionIndex}/{total_questions}"
- Treść pytania (nadal widoczna)
- 4 przyciski odpowiedzi: A, B, C, D
  - Duże (min 60px wysokości), różne kolory
  - Mobile-friendly, łatwe do kliknięcia
- Timer: countdown 20, 19, 18...0
- Pasek postępu (progress bar) wizualizujący pozostały czas

**Interakcja - gracz klika odpowiedź:**
1. Obliczenie `time_taken_ms` (timestamp kliknięcia - timestamp pokazania odpowiedzi)
2. Przycisk disabled + loading state
3. POST /api/game/:sessionId `{action: 'submit_answer', question_id, answer_index, time_taken_ms}`
   - Backend:
     - Sprawdzenie is_correct (porównanie z answers.is_correct)
     - Obliczenie points (formuła poniżej)
     - Zapis do game_answers: game_session_id, question_id, answer_index, is_correct, time_taken_ms, points_earned
     - Update game_sessions.total_points
     - Sprawdzenie czy to ostatnie pytanie
   - Response: `{is_correct, correct_answer_index, points_earned, total_points, is_last_question: boolean}`
4. Zapisanie response w state
5. `setQuestionSubState('feedback')`

**Jeśli brak odpowiedzi w 20s:**
- Automatyczne wywołanie POST /api/game/:sessionId z `{action: 'submit_answer', question_id, answer_index: null, time_taken_ms: 20000}`
- Backend zapisuje: is_correct=false, points_earned=0

**Formuła punktacji:**
```javascript
MAX_POINTS = 1000
TIME_LIMIT_MS = 20000

if (is_correct) {
  points = MAX_POINTS - Math.round(time_taken_ms / 20)
  points = Math.max(0, points) // minimum 0, maximum 1000
} else {
  points = 0
}
```

**Przykłady:**
- Odpowiedź poprawna po 100ms: 1000 - 5 = 995 pkt
- Odpowiedź poprawna po 5000ms: 1000 - 250 = 750 pkt
- Odpowiedź poprawna po 10000ms: 1000 - 500 = 500 pkt
- Odpowiedź poprawna po 20000ms: 1000 - 1000 = 0 pkt
- Odpowiedź błędna: 0 pkt
- Brak odpowiedzi: 0 pkt

---

3.3.5 Pytanie - feedback (3 sekundy)
**State:** `gameState = 'question'`, `questionSubState = 'feedback'`

**Dane z API:** `{is_correct, correct_answer_index, points_earned}`

**Wyświetlenie - jeśli poprawna odpowiedź:**
- Zielone tło (bg-green-500, full screen)
- Ikona ✓ (duża, wyśrodkowana)
- Tekst: "Świetnie!" lub "Dobra odpowiedź!"
- "+{points_earned} pkt" (np. "+950 pkt")

**Wyświetlenie - jeśli błędna odpowiedź:**
- Czerwone tło (bg-red-500, full screen)
- Ikona ✗ (duża, wyśrodkowana)
- Tekst: "Błąd!" lub "Niestety źle"
- "0 pkt"
- "Prawidłowa odpowiedź: {letter}" (A, B, C lub D z correct_answer_index)

**Po 3 sekundach (setTimeout):**
- Sprawdzenie flagi `is_last_question` z response
- Jeśli `is_last_question === false`:
  - `setCurrentQuestionIndex(prev => prev + 1)`
  - `setQuestionSubState('reading')`
  - Fetch następnego pytania: POST /api/game/:sessionId `{action: 'get_question', question_index: nextIndex}`
  - Powrót do fazy czytania pytania
- Jeśli `is_last_question === true`:
  - `setGameState('results')`
  - Backend już ustawił status='completed' przy ostatniej odpowiedzi
  - Fetch wyników: POST /api/game/:sessionId `{action: 'get_results'}`

---

3.3.6 Ekran wyników końcowych (z TOP 10)
**State:** `gameState = 'results'`

**Fetch danych:**
- POST /api/game/:sessionId `{action: 'get_results'}`
- Response: `{player_nickname, total_points, correct_answers, total_questions, rank, percentile, leaderboard_top10: [{rank, player_nickname, total_points}]}`

**Wyświetlenie - sekcja "Twoje wyniki":**
- Komunikat: "Koniec quizu! 🎉"
- Card z wynikami gracza (wyróżniony wizualnie):
  - Pseudonim: "{player_nickname}"
  - Łączne punkty: "{total_points} pkt" (duża czcionka)
  - Poprawne odpowiedzi: "{correct_answers}/{total_questions}"
  - Procent poprawnych: "{percentage}%" (np. "70%")
  - Twoja pozycja: "{rank}. miejsce"
  - Percentile: "Lepszy niż {percentile}% graczy"

**Wyświetlenie - sekcja "Najlepsi gracze":**
- Header: "TOP 10" lub "Ranking"
- Tabela/Lista TOP 10 graczy:
  - Kolumny: Pozycja | Pseudonim | Punkty
  - Sortowanie: total_points DESC
  - TOP 3 z ikonami podium: 🥇 🥈 🥉
  - Wyróżnienie wizualne TOP 3 (kolor tła, większa czcionka)
  - Highlight aktualnego gracza jeśli jest w TOP 10 (border, inne tło)
- Jeśli aktualny gracz nie jest w TOP 10:
  - Separator "..."
  - Wyświetlenie pozycji gracza: "{rank}. | {nickname} | {points} pkt" (highlight)

**Przyciski akcji:**
- "Zagraj ponownie"
  - Przekierowanie do `/play/:pin`
  - Nowy game_session (pełny reset stanu)
  - Możliwość poprawy wyniku
- "Strona główna"
  - Przekierowanie do `/`

**Layout:**
- Mobile: vertical stack (wyniki gracza → TOP 10)
- Desktop: możliwe 2 kolumny (wyniki gracza po lewej, TOP 10 po prawej)

**Animacje (opcjonalnie):**
- Fade in dla wyników gracza
- Stagger animation dla TOP 10 (każdy item po kolei)
- Confetti effect jeśli gracz w TOP 3 🎉

### 3.4 Baza danych (Supabase PostgreSQL)

Struktura tabel:

```sql
-- Użytkownicy (twórcy quizów)
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email TEXT UNIQUE NOT NULL,
  nickname TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
-- Uwaga: hasło zarządzane przez Supabase Auth (auth.users)

-- Quizy
CREATE TABLE quizzes (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  pin TEXT UNIQUE NOT NULL, -- 6-cyfrowy PIN
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Pytania
CREATE TABLE questions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  quiz_id UUID NOT NULL REFERENCES quizzes(id) ON DELETE CASCADE,
  question_text TEXT NOT NULL,
  order_index INTEGER NOT NULL, -- kolejność pytań (1, 2, 3...)
  created_at TIMESTAMP DEFAULT NOW()
);

-- Odpowiedzi
CREATE TABLE answers (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  question_id UUID NOT NULL REFERENCES questions(id) ON DELETE CASCADE,
  answer_text TEXT NOT NULL,
  is_correct BOOLEAN NOT NULL DEFAULT FALSE,
  order_index INTEGER NOT NULL, -- 0=A, 1=B, 2=C, 3=D
  created_at TIMESTAMP DEFAULT NOW()
);

-- Sesje gry (rozgrywki)
CREATE TABLE game_sessions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  quiz_id UUID NOT NULL REFERENCES quizzes(id) ON DELETE CASCADE,
  player_nickname TEXT NOT NULL,
  total_points INTEGER DEFAULT 0,
  status TEXT NOT NULL DEFAULT 'in_progress', -- 'in_progress', 'completed', 'abandoned'
  started_at TIMESTAMP DEFAULT NOW(),
  completed_at TIMESTAMP,
  CONSTRAINT valid_status CHECK (status IN ('in_progress', 'completed', 'abandoned'))
);

-- Odpowiedzi graczy
CREATE TABLE game_answers (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  game_session_id UUID NOT NULL REFERENCES game_sessions(id) ON DELETE CASCADE,
  question_id UUID NOT NULL REFERENCES questions(id) ON DELETE CASCADE,
  answer_index INTEGER NOT NULL, -- 0=A, 1=B, 2=C, 3=D
  is_correct BOOLEAN NOT NULL,
  time_taken_ms INTEGER NOT NULL,
  points_earned INTEGER NOT NULL DEFAULT 0,
  answered_at TIMESTAMP DEFAULT NOW()
);

-- Indeksy dla wydajności
CREATE INDEX idx_quizzes_user_id ON quizzes(user_id);
CREATE INDEX idx_quizzes_pin ON quizzes(pin);
CREATE INDEX idx_questions_quiz_id ON questions(quiz_id);
CREATE INDEX idx_answers_question_id ON answers(question_id);
CREATE INDEX idx_game_sessions_quiz_id ON game_sessions(quiz_id);
CREATE INDEX idx_game_sessions_status ON game_sessions(status);
CREATE INDEX idx_game_answers_session_id ON game_answers(game_session_id);
```

Row Level Security (RLS) policies:
```sql
-- Użytkownicy widzą tylko swoje quizy
CREATE POLICY "Users can view own quizzes"
  ON quizzes FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own quizzes"
  ON quizzes FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own quizzes"
  ON quizzes FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete own quizzes"
  ON quizzes FOR DELETE
  USING (auth.uid() = user_id);

-- Wszyscy (nawet niezalogowani) mogą czytać pytania i odpowiedzi po PIN
CREATE POLICY "Anyone can view questions"
  ON questions FOR SELECT
  USING (true);

CREATE POLICY "Anyone can view answers"
  ON answers FOR SELECT
  USING (true);

-- Wszyscy mogą tworzyć game_sessions (anonimowi gracze)
CREATE POLICY "Anyone can create game sessions"
  ON game_sessions FOR INSERT
  WITH CHECK (true);

CREATE POLICY "Anyone can view game sessions"
  ON game_sessions FOR SELECT
  USING (true);

-- Gracze mogą zapisywać swoje odpowiedzi
CREATE POLICY "Anyone can insert game answers"
  ON game_answers FOR INSERT
  WITH CHECK (true);
```

### 3.5 Strony i routing

Routes:
- `/` - strona główna (input PIN + "Rozpocznij quiz", link "Zarejestruj się")
- `/register` - rejestracja użytkownika
- `/login` - logowanie użytkownika
- `/dashboard` - dashboard użytkownika z listą quizów (wymaga auth)
- `/quiz/new` - tworzenie nowego quizu (wymaga auth)
- `/quiz/:id/edit` - edycja quizu (wymaga auth)
- `/quiz/:id/results` - analytics i leaderboard dla HOST-a (wymaga auth + ownership)
- `/play/:pin` - CAŁY FLOW GRY w jednym route (React component zarządza wszystkim)

Uwaga: Route `/play/:pin` renderuje jeden React component (QuizPlay) który dynamicznie wyświetla:
1. Formularz z pseudonimem (jeśli sesja nie istnieje)
2. Countdown (10s) 
3. Pytanie po pytaniu (5s bez odpowiedzi + 20s z odpowiedziami + 3s feedback)
4. Wyniki końcowe

Stan gry zarządzany przez React:
- `gameState`: 'nickname_input' | 'countdown' | 'question' | 'results'
- `sessionId` - zwracany z API po POST /api/game/start, trzymany w state
- `currentQuestionIndex` - aktualny numer pytania
- `questionData` - dane aktualnego pytania
- `questionSubState` - dla gameState='question': 'reading' (5s) | 'answering' (20s) | 'feedback' (3s)
- `totalPoints` - suma punktów
- `quizInfo` - {quiz_id, title, total_questions}

Wszystkie przejścia między etapami obsługiwane przez React state management (nie przez routing)

Przykładowa struktura komponentu QuizPlay:
```typescript
function QuizPlay() {
  const { pin } = useParams();
  const [gameState, setGameState] = useState('nickname_input');
  const [sessionId, setSessionId] = useState(null);
  const [quizInfo, setQuizInfo] = useState(null);
  const [currentQuestionIndex, setCurrentQuestionIndex] = useState(0);
  const [questionData, setQuestionData] = useState(null);
  const [questionSubState, setQuestionSubState] = useState('reading');
  const [totalPoints, setTotalPoints] = useState(0);
  
  // Conditional rendering based on gameState
  if (gameState === 'nickname_input') return <NicknameForm />;
  if (gameState === 'countdown') return <Countdown />;
  if (gameState === 'question') return <QuestionView />;
  if (gameState === 'results') return <Results />;
}
```

API Routes (Astro API):

Autentykacja:
- `POST /api/auth/register` - rejestracja użytkownika
- `POST /api/auth/login` - logowanie użytkownika
- `POST /api/auth/logout` - wylogowanie użytkownika

Zarządzanie quizami:
- `GET /api/quizzes` - lista quizów użytkownika (wymaga auth)
- `POST /api/quizzes` - tworzenie quizu (wymaga auth)
- `GET /api/quizzes/:id` - szczegóły quizu (wymaga auth)
- `PUT /api/quizzes/:id` - edycja quizu (wymaga auth)
- `DELETE /api/quizzes/:id` - usunięcie quizu (wymaga auth)
- `GET /api/quizzes/:id/analytics?filter=all|24h|7d` - statystyki i leaderboard dla HOST-a (wymaga auth)
  Response: `{quiz_title, total_games, completed_games, abandoned_games, average_score, leaderboard: [...]}`
- `POST /api/ai/generate` - generowanie pytań przez AI (wymaga auth)

Rozgrywka (publiczne, bez auth):
- `GET /api/quiz/:pin` - pobranie podstawowych info o quizie (tytuł, liczba pytań)
  Response: `{quiz_id, title, total_questions}`

- `POST /api/game/start` - utworzenie sesji gry
  Request: `{quiz_id, player_nickname}`
  Response: `{session_id, quiz_title, total_questions}`

- `POST /api/game/:sessionId` - UNIWERSALNY endpoint zarządzający całym flow gry
  
  **Pobranie pytania:**
  Request: `{action: 'get_question', question_index: number}`
  Response: `{question_id, question_text, answers: [{text, order_index}], question_number, total_questions}`
  
  **Submit odpowiedzi:**
  Request: `{action: 'submit_answer', question_id, answer_index, time_taken_ms}`
  Response: `{is_correct, correct_answer_index, points_earned, total_points, is_last_question: boolean}`
  - Backend zapisuje odpowiedź w game_answers
  - Oblicza punktację
  - Update game_sessions.total_points
  - Jeśli ostatnie pytanie: update status='completed'
  
  **Pobranie wyników końcowych:**
  Request: `{action: 'get_results'}`
  Response: `{player_nickname, total_points, correct_answers, total_questions, rank, percentile, leaderboard_top10: [{rank, player_nickname, total_points}]}`

### 3.6 Responsywność i UX

Priorytety urządzeń:
- **Mobile-first** (większość graczy używa telefonów)
- Desktop dla twórców quizów (dashboard, formularz)

Breakpointy:
- Mobile: < 768px
- Tablet: 768px - 1024px
- Desktop: > 1024px

Wymagania mobile (dla graczy):
- Przyciski odpowiedzi: minimum 60px wysokości (łatwe do kliknięcia)
- Font size: minimum 16px (zapobiega auto-zoom na iOS)
- Full-screen mode podczas gry
- Vertical layout
- Touch-friendly interactions

Desktop (dla twórców):
- Dashboard: table/grid view dla listy quizów
- Formularz tworzenia: responsywny layout
- 2-column layout na dużych ekranach

Kolory i typografia:
```css
:root {
  --primary: #6366f1; /* Indigo */
  --primary-hover: #4f46e5;
  --success: #22c55e; /* Green */
  --error: #ef4444; /* Red */
  --background: #ffffff;
  --text-primary: #111827;
}
```

Komponenty (Shadcn/ui):
- Button, Input, Textarea
- Card, Dialog/Modal
- Toast notifications
- Progress bar (timer)

### 3.7 Error handling

Kluczowe scenariusze błędów:

1. **Nieprawidłowy PIN:**
   - "Quiz o podanym PIN nie istnieje"
   - Możliwość wpisania innego PIN

2. **AI generowanie niepowodzenie:**
   - Timeout: "Generowanie trwa zbyt długo, spróbuj ponownie"
   - Błąd API: "Nie udało się wygenerować pytań. Spróbuj dodać więcej treści."
   - Rate limit: "Osiągnięto dzienny limit generacji (5). Spróbuj jutro."

3. **Upload zbyt dużego pliku:**
   - "Plik jest zbyt duży. Maksymalny rozmiar: 10 MB (PDF) / 5 MB (zdjęcia)"

4. **Błąd zapisu do bazy:**
   - "Nie udało się zapisać. Spróbuj ponownie."
   - Retry button

5. **Session expired:**
   - Automatyczne przekierowanie do /login
   - "Sesja wygasła. Zaloguj się ponownie."

6. **Network error podczas gry:**
   - Retry automatyczny (3 próby)
   - Jeśli fail: "Utracono połączenie. Sprawdź internet i odśwież stronę."
   - Local storage backup dla odpowiedzi (recovery po refresh)

## 4. Granice produktu

Funkcjonalności NIE wchodzące w zakres MVP:

Funkcje użytkowników:
- Personalizowane profile użytkowników z avatarami
- Social login (Google, Facebook, Apple)
- Email confirmation przy rejestracji
- Reset hasła / Forgot password
- Zmiana email lub hasła

Zaawansowane zarządzanie quizami:
- Udostępnianie quizów publicznie (katalog publicznych quizów)
- Komentarze i oceny quizów
- Duplikowanie quizów
- Export/import quizów (JSON, CSV)
- Kategoryzacja i tagi
- Drag & drop dla zmiany kolejności pytań
- Pytania z obrazkami/wideo
- Różne typy pytań (prawda/fałsz, multi-select, pytania otwarte)

Zaawansowana rozgrywka:
- Tryb real-time multiplayer (wszyscy grają jednocześnie jak w Kahoot)
- Tryb drużynowy
- Power-upy lub bonusy
- Streak bonusy za serie poprawnych odpowiedzi
- Możliwość pauzy w trakcie gry
- Quiz z limitem czasowym dla całego quizu (nie tylko pytań)

Analytics i raporty:
- Szczegółowe statystyki dla twórców (które pytania najtrudniejsze, średni czas)
- Export wyników do PDF/CSV
- Wykresy i wizualizacje
- Tracking postępów indywidualnych graczy w czasie
- A/B testing różnych wersji pytań

Integracje:
- Integracja z LMS (Google Classroom, Moodle)
- Integracja z zewnętrznymi źródłami wiedzy (Wikipedia, YouTube)
- API dla developerów
- Webhooks

Gamification:
- System nagród i odznak (achievements)
- Rankingi globalne (nie tylko per quiz)
- Poziomy graczy i experience points
- Daily challenges
- Profile graczy z historią (wymagałoby rejestracji graczy)

Social features:
- Udostępnianie wyników na social media
- Challenges między użytkownikami
- Chat w trakcie quizu
- Możliwość "followowania" twórców quizów

Monetyzacja:
- Płatne plany / Freemium
- Reklamy
- White-label dla firm
- Premium features (więcej pytań, nielimitowane AI generation)

Inne:
- Multi-language support (i18n)
- Dark mode
- Offline mode
- Mobilna aplikacja natywna (iOS/Android)
- Dostępność (screen reader, high contrast mode)
- Custom branding dla organizacji
- QR code generation dla łatwiejszego udostępniania

Ograniczenia techniczne MVP:
- Maksymalna liczba pytań w quizie: 20
- Rate limiting AI generation: 5 generacji/dzień/user
- Brak backupu game_sessions starszych niż 90 dni
- Brak soft delete (permanent delete)

## 5. Historyjki użytkowników

### 5.1 Autentykacja i autoryzacja

US-001
Tytuł: Rejestracja użytkownika (twórcy quizu)
Opis: Jako nowy użytkownik chcę zarejestrować się w aplikacji za pomocą email i hasła, aby móc tworzyć i zarządzać swoimi quizami.
Kryteria akceptacji:
- Formularz rejestracji dostępny pod /register
- Pola: email (format email), hasło (min. 8 znaków), pseudonim (3-30 znaków)
- Walidacja unikalności email w systemie
- Supabase Auth automatycznie hashuje hasło
- Po sukcesie: automatyczne zalogowanie i przekierowanie do /dashboard
- Błędy: "Email jest już zajęty", "Hasło musi mieć minimum 8 znaków", "Pseudonim jest wymagany"
- Toast notification: "Witaj! Twoje konto zostało utworzone"

US-002
Tytuł: Logowanie użytkownika
Opis: Jako zarejestrowany użytkownik chcę zalogować się do aplikacji, aby uzyskać dostęp do moich quizów i dashboardu.
Kryteria akceptacji:
- Formularz logowania dostępny pod /login
- Pola: email, hasło
- Weryfikacja credentials przez Supabase Auth
- Po sukcesie: przekierowanie do /dashboard
- Sesja przechowywana w JWT token (Supabase)
- Błąd: "Nieprawidłowy email lub hasło" przy błędnych danych
- Link "Nie masz konta? Zarejestruj się"

US-003
Tytuł: Wylogowanie użytkownika
Opis: Jako zalogowany użytkownik chcę wylogować się z aplikacji, aby zabezpieczyć swoje konto.
Kryteria akceptacji:
- Przycisk "Wyloguj" w header dla zalogowanych użytkowników
- Kliknięcie wywołuje Supabase Auth signOut()
- Usunięcie JWT token
- Przekierowanie do strony głównej (/)
- Toast notification: "Zostałeś wylogowany"

US-004
Tytuł: Ochrona dostępu do stron dla zalogowanych
Opis: Jako system chcę zabezpieczyć strony wymagające autoryzacji, aby tylko zalogowani użytkownicy mieli do nich dostęp.
Kryteria akceptacji:
- Middleware sprawdza JWT token dla routes: /dashboard, /quiz/new, /quiz/:id/edit
- Niezalogowani użytkownicy są automatycznie przekierowani do /login
- Zalogowani użytkownicy mają pełny dostęp
- Middleware działa na poziomie Astro middleware lub API routes

### 5.2 Tworzenie i zarządzanie quizami

US-005
Tytuł: Wyświetlenie dashboardu z listą quizów
Opis: Jako zalogowany użytkownik chcę zobaczyć listę wszystkich moich quizów, aby wybrać który chcę edytować, udostępnić lub zobaczyć wyniki.
Kryteria akceptacji:
- Dashboard dostępny pod /dashboard (wymaga auth)
- Lista quizów z informacjami: tytuł, liczba pytań, PIN, data utworzenia, liczba rozegranych gier
- Sortowanie: najnowsze na górze
- Przycisk "Kopiuj PIN" przy każdym quizie (clipboard API)
- Akcje: "Edytuj", "Zobacz wyniki", "Usuń"
- Przycisk CTA: "Stwórz nowy quiz" (prowadzi do /quiz/new)
- Empty state: "Nie masz jeszcze żadnych quizów. Stwórz pierwszy!" + ilustracja

US-006
Tytuł: Ręczne tworzenie quizu
Opis: Jako zalogowany użytkownik chcę stworzyć quiz manualnie, dodając pytania i odpowiedzi, aby mieć pełną kontrolę nad treścią.
Kryteria akceptacji:
- Formularz dostępny pod /quiz/new (wymaga auth)
- Pole "Nazwa quizu" (required, max 100 znaków)
- Dynamiczna lista pytań z przyciskiem "Dodaj pytanie"
- Każde pytanie: textarea dla treści (max 500 znaków), 4 inputy dla odpowiedzi, checkboxy dla prawidłowych
- Walidacja: min 3 pytania, max 20, każde pytanie 4 odpowiedzi, min 1 prawidłowa
- Przycisk "Zapisz quiz i wygeneruj PIN"
- Po zapisie: automatyczne generowanie unikalnego 6-cyfrowego PIN
- Przekierowanie do /dashboard
- Toast: "Quiz został zapisany! PIN: {pin}"

US-007
Tytuł: Generowanie quizu za pomocą AI
Opis: Jako zalogowany użytkownik chcę wygenerować quiz automatycznie z materiałów (tekst, PDF, zdjęcie) za pomocą AI, aby zaoszczędzić czas.
Kryteria akceptacji:
- Sekcja "AI Generator" na górze formularza /quiz/new
- Textarea dla instrukcji: "Opisz quiz (temat, poziom, liczba pytań)"
- Input type=file dla uploadu (PDF max 10MB, JPG/PNG max 5MB)
- Przycisk "Generuj pytania"
- Loading state: spinner + "Generowanie pytań... To może potrwać do 60 sekund"
- Request do /api/ai/generate (OpenRouter GPT-4)
- Po sukcesie: formularz wypełniony wygenerowanymi pytaniami
- Użytkownik może edytować przed zapisaniem
- Pliki NIE są zapisywane (tylko do analizy)
- Rate limiting: 5 generacji/dzień/user
- Błędy:
  - Timeout: "Generowanie trwa zbyt długo, spróbuj ponownie"
  - API error: "Nie udało się wygenerować pytań..."
  - Rate limit: "Osiągnięto dzienny limit (5). Spróbuj jutro."
  - Plik za duży: "Plik jest zbyt duży..."

US-008
Tytuł: Edycja zapisanego quizu
Opis: Jako zalogowany użytkownik chcę edytować swój quiz, aby poprawić pytania lub dostosować treść.
Kryteria akceptacji:
- Przycisk "Edytuj" w dashboardzie prowadzi do /quiz/:id/edit
- Tylko właściciel może edytować (sprawdzenie user_id = auth.uid())
- Formularz wypełniony danymi quizu
- Brak sekcji AI Generator (tylko manualna edycja)
- Można dodawać/usuwać pytania, edytować treści
- Walidacja identyczna jak przy tworzeniu
- PIN pozostaje ten sam po edycji
- Przycisk "Zapisz zmiany"
- Toast: "Zmiany zostały zapisane"

US-009
Tytuł: Usuwanie quizu
Opis: Jako zalogowany użytkownik chcę usunąć quiz, aby usunąć niepotrzebny quiz i związane z nim wyniki.
Kryteria akceptacji:
- Przycisk "Usuń" przy każdym quizie w dashboardzie
- Modal konfirmacji: "Czy na pewno chcesz usunąć quiz '{title}'? Zostaną usunięte również wszystkie wyniki gier. Tej operacji nie można cofnąć."
- Przyciski: "Anuluj" (zamyka modal), "Usuń" (kasuje quiz)
- Cascade delete: quiz + questions + answers + game_sessions + game_answers
- Odświeżenie listy quizów
- Toast: "Quiz został usunięty"

US-010
Tytuł: Kopiowanie PIN do schowka
Opis: Jako zalogowany użytkownik chcę skopiować PIN quizu jednym kliknięciem, aby łatwo udostępnić go innym.
Kryteria akceptacji:
- Przycisk "Kopiuj PIN" przy każdym quizie w dashboardzie
- Kliknięcie kopiuje 6-cyfrowy PIN do clipboard (Clipboard API)
- Wizualny feedback: ikona zmienia się na checkmark
- Toast: "PIN skopiowany: {pin}"
- Po 2 sekundach ikona wraca do stanu początkowego

US-011
Tytuł: Podgląd analytics i leaderboard quizu przez HOST-a
Opis: Jako twórca quizu chcę zobaczyć statystyki i wszystkie wyniki graczy mojego quizu, aby monitorować jak ludzie radzą sobie z moim quizem.
Kryteria akceptacji:
- Przycisk "Zobacz wyniki" w dashboardzie prowadzi do /quiz/:id/results (wymaga auth + ownership)
- GET /api/quizzes/:id/analytics
- Sekcja statystyk (karty):
  - Liczba rozegranych gier (total_games)
  - Liczba ukończonych gier (completed_games) + percentage
  - Liczba porzuconych gier (abandoned_games)
  - Średni wynik (average_score pkt)
  - Średni procent poprawnych odpowiedzi
- Sekcja leaderboard:
  - Dropdown filtrowania: "Wszystkie czasy" / "Ostatnie 24h" / "Ostatni tydzień"
  - Tabela wszystkich graczy: Pozycja | Pseudonim | Punkty | Poprawne | % | Kiedy
  - Sortowanie: total_points DESC, completed_at ASC
  - TOP 3 z podium 🥇🥈🥉
  - Paginacja: 20 wyników na stronę
  - Search: wyszukiwanie po pseudonimie
- Link "Udostępnij PIN: {pin}" z przyciskiem "Kopiuj"
- Przyciski: "Edytuj quiz", "Wróć do dashboardu"
- Przycisk "Odśwież" do reload danych
- Empty state: "Nikt jeszcze nie rozwiązał tego quizu. Udostępnij PIN: {pin}"

### 5.3 Rozpoczęcie gry przez gracza

US-012
Tytuł: Wpisanie PIN i rozpoczęcie quizu
Opis: Jako gracz (anonimowy) chcę wpisać 6-cyfrowy PIN na stronie głównej, aby rozpocząć quiz.
Kryteria akceptacji:
- Strona główna (/) zawiera duży input dla PIN (6 cyfr)
- Przycisk "Rozpocznij quiz"
- Input akceptuje tylko cyfry 0-9
- Przycisk disabled gdy PIN != 6 cyfr
- Po kliknięciu: sprawdzenie czy quiz o danym PIN istnieje (GET /api/quiz/:pin)
- Jeśli istnieje: przekierowanie do /play/:pin
- Jeśli nie: błąd "Quiz o podanym PIN nie istnieje"
- Możliwość ponownego wpisania PIN

US-013
Tytuł: Podanie pseudonimu przed grą
Opis: Jako gracz chcę podać pseudonim przed rozpoczęciem gry, aby moje wyniki były widoczne w leaderboardzie.
Kryteria akceptacji:
- Route /play/:pin ładuje komponent QuizPlay
- Początkowy gameState: 'nickname_input'
- useEffect pobiera info o quizie: GET /api/quiz/:pin
  Response: {quiz_id, title, total_questions}
- Wyświetlenie:
  - Tytuł quizu
  - Liczba pytań: "{total_questions} pytań"
  - Input dla pseudonimu (3-30 znaków, required)
  - Przycisk "Start"
- Walidacja: pseudonim 3-30 znaków
- Po kliknięciu "Start": 
  - POST /api/game/start {quiz_id, player_nickname}
  - Utworzenie game_session w bazie (quiz_id, player_nickname, status='in_progress', started_at=NOW())
  - Response: {session_id, quiz_title, total_questions}
  - Zapisanie session_id w React state
  - Zmiana gameState na 'countdown'

US-014
Tytuł: Countdown 10 sekund przed grą
Opis: Jako gracz chcę zobaczyć countdown przed rozpoczęciem quizu, aby przygotować się do gry.
Kryteria akceptacji:
- gameState: 'countdown' (po utworzeniu sesji)
- Wyświetlenie:
  - Tytuł quizu
  - "{total_questions} pytań"
  - "Zaczynamy za... 10, 9, 8, 7, 6, 5, 4, 3, 2, 1, START!"
- Duża czcionka, animacja liczb
- Countdown client-side (React useState + useEffect z setInterval)
- Po 10 sekundach: 
  - Zmiana gameState na 'question'
  - currentQuestionIndex = 1
  - Fetch pierwszego pytania: POST /api/game/:sessionId `{action: 'get_question', question_index: 1}`

### 5.4 Rozgrywka pytania

US-015
Tytuł: Wyświetlenie pytania bez odpowiedzi (5 sekund)
Opis: Jako gracz chcę zobaczyć treść pytania przez 5 sekund przed pokazaniem odpowiedzi, aby mieć czas na przeczytanie i zrozumienie.
Kryteria akceptacji:
- Stan komponentu: 'question' z sub-stanem 'reading' (pierwsze 5s)
- Pobranie pytania: POST /api/game/:sessionId `{action: 'get_question', question_index: currentQuestionIndex}`
- Response: `{question_id, question_text, answers: [{text, order_index}], question_number, total_questions}`
- Wyświetlenie:
  - Numer pytania: "Pytanie {question_number}/{total_questions}"
  - Treść pytania (duża czcionka, wyśrodkowana)
  - Komunikat: "Odpowiedzi pojawią się za..."
  - Timer: countdown 5, 4, 3, 2, 1 (React state)
- Przyciski odpowiedzi są UKRYTE (conditional rendering)
- Po 5 sekundach: zmiana sub-stanu na 'answering' + pokazanie przycisków + start 20s timera

US-016
Tytuł: Odpowiadanie na pytanie (20 sekund)
Opis: Jako gracz chcę wybrać odpowiedź z 4 opcji w ciągu 20 sekund, aby zdobyć punkty.
Kryteria akceptacji:
- Sub-stan: 'answering' (po 5s czytania)
- Pokazanie 4 przycisków: A, B, C, D (conditional rendering)
- Duże przyciski (min 60px wysokości), różne kolory, łatwe do kliknięcia
- Timer: countdown 20, 19, 18...0 (React state + useEffect)
- Pasek postępu wizualizujący pozostały czas (progress bar component)
- Gracz klika jeden przycisk:
  - Obliczenie time_taken_ms (timestamp kliknięcia - timestamp pokazania odpowiedzi)
  - Przycisk disabled + loading state
  - POST /api/game/:sessionId `{action: 'submit_answer', question_id, answer_index, time_taken_ms}`
  - Backend sprawdza is_correct, oblicza points, zapisuje do game_answers, update game_sessions.total_points
  - Response: `{is_correct, correct_answer_index, points_earned, total_points, is_last_question: boolean}`
  - Zapisanie response w state
- Jeśli brak odpowiedzi w 20s:
  - Automatyczne wywołanie POST /api/game/:sessionId `{action: 'submit_answer', question_id, answer_index: null, time_taken_ms: 20000}`
  - Backend zapisuje: 0 punktów
- Zmiana sub-stanu na 'feedback' z danymi z response

US-017
Tytuł: Feedback po odpowiedzi na pytanie
Opis: Jako gracz chcę zobaczyć czy odpowiedziałem poprawnie i ile punktów zdobyłem, aby otrzymać natychmiastowy feedback.
Kryteria akceptacji:
- Sub-stan: 'feedback' z danymi `{is_correct, correct_answer_index, points_earned, total_points, is_last_question}`
- Wyświetlenie przez 3 sekundy (setTimeout)
- Jeśli poprawna odpowiedź:
  - Zielone tło ekranu (bg-green-500)
  - Ikona ✓ (duża)
  - Tekst: "Świetnie!" lub "Dobra odpowiedź!"
  - "+{points_earned} pkt" (np. "+950 pkt")
- Jeśli błędna odpowiedź:
  - Czerwone tło ekranu (bg-red-500)
  - Ikona ✗ (duża)
  - Tekst: "Błąd!" lub "Niestety źle"
  - "0 pkt"
  - "Prawidłowa odpowiedź: {letter}" (A, B, C lub D z correct_answer_index)
- Po 3 sekundach:
  - Sprawdzenie flagi `is_last_question` z response
  - Jeśli `is_last_question === false`: 
    - currentQuestionIndex++
    - Zmiana sub-stanu na 'reading'
    - Fetch następnego pytania: POST /api/game/:sessionId `{action: 'get_question', question_index: nextIndex}`
  - Jeśli `is_last_question === true`:
    - Zmiana gameState na 'results'
    - Fetch wyników: POST /api/game/:sessionId `{action: 'get_results'}`

US-018
Tytuł: Obliczanie punktacji za odpowiedź
Opis: Jako system chcę obliczyć punkty dla gracza na podstawie poprawności i czasu odpowiedzi, aby nagradzać szybkie i prawidłowe odpowiedzi.
Kryteria akceptacji:
- Formuła: points = 1000 - Math.round(time_taken_ms / 20)
- Jeśli odpowiedź błędna: 0 punktów
- Jeśli brak odpowiedzi: 0 punktów
- Punkty zaokrąglone do całości
- Minimum: 0 punktów (nie może być ujemnych)
- Przykłady:
  - 100ms (poprawna) = 1000 - 5 = 995 pkt
  - 5000ms (poprawna) = 1000 - 250 = 750 pkt
  - 20000ms (poprawna) = 1000 - 1000 = 0 pkt
  - Błędna = 0 pkt
- Zapis do game_answers.points_earned
- Update game_sessions.total_points (suma wszystkich points_earned)

### 5.5 Wyniki i leaderboard

US-019
Tytuł: Wyświetlenie wyników końcowych i TOP 10 po zakończeniu quizu
Opis: Jako gracz chcę zobaczyć swoje wyniki oraz TOP 10 najlepszych graczy po zakończeniu quizu, aby wiedzieć jak sobie poradziłem i porównać się z innymi.
Kryteria akceptacji:
- Stan komponentu: gameState='results'
- Sesja ma status='completed' (ustawione przy ostatniej odpowiedzi)
- Fetch: POST /api/game/:sessionId `{action: 'get_results'}`
  Response: `{player_nickname, total_points, correct_answers, total_questions, rank, percentile, leaderboard_top10: [{rank, player_nickname, total_points}]}`
- Wyświetlenie sekcji "Twoje wyniki":
  - "Koniec quizu! 🎉"
  - Card z wynikami (wyróżniony wizualnie):
    - Pseudonim
    - Łączne punkty: "{total_points} pkt" (duża czcionka)
    - Poprawne odpowiedzi: "{correct_answers}/{total_questions}"
    - Procent poprawnych: "{percentage}%"
    - Pozycja: "{rank}. miejsce"
    - Percentile: "Lepszy niż {percentile}% graczy"
- Wyświetlenie sekcji "TOP 10":
  - Header: "Najlepsi gracze" lub "Ranking"
  - Lista TOP 10 z podium 🥇🥈🥉 dla pierwszych trzech
  - Highlight aktualnego gracza jeśli w TOP 10
  - Jeśli poza TOP 10: separator "..." + pozycja gracza
- Layout: mobile vertical, desktop 2 kolumny (opcjonalnie)
- Przyciski:
  - "Zagraj ponownie" → /play/:pin
  - "Strona główna" → /
- Opcjonalne animacje: fade in, stagger, confetti dla TOP 3

US-020
Tytuł: Możliwość ponownego rozegrania quizu
Opis: Jako gracz który ukończył quiz chcę móc zagrać ponownie, aby poprawić swój wynik.
Kryteria akceptacji:
- Przycisk "Zagraj ponownie" na ekranie wyników (gameState='results')
- Kliknięcie przekierowuje do /play/:pin
- Pełny reset stanu React (nowy component mount)
- Tworzenie NOWEGO game_session przez POST /api/game/start
- Każda rozgrywka jest osobnym rekordem w bazie (game_sessions)
- Gracz może mieć wiele sesji dla tego samego quizu
- W TOP 10 może pojawić się ten sam pseudonim wielokrotnie (jeśli różne sesje)
- Każda sesja ma unikalny session_id

### 5.6 Edge cases i walidacja

US-021
Tytuł: Obsługa nieprawidłowego PIN
Opis: Jako gracz który wpisał nieprawidłowy PIN chcę otrzymać jasny komunikat błędu.
Kryteria akceptacji:
- Wpisanie 6-cyfrowego PIN który nie istnieje w bazie
- GET /api/quiz/:pin zwraca 404 Not Found
- Wyświetlenie błędu: "Quiz o podanym PIN nie istnieje"
- Input PIN zostaje wyczyszczony lub focus
- Możliwość wpisania innego PIN
- Sugestia: "Sprawdź czy PIN jest poprawny"

US-022
Tytuł: Obsługa timeout podczas generowania AI
Opis: Jako użytkownik który generuje quiz przez AI chcę być poinformowany jeśli generowanie trwa zbyt długo.
Kryteria akceptacji:
- Request do OpenRouter API ma timeout 60 sekund
- Jeśli >60s: przerwanie requestu
- Wyświetlenie błędu: "Generowanie trwa zbyt długo. Spróbuj ponownie lub zmień materiał na krótszy."
- Przycisk "Spróbuj ponownie"
- Formularz NIE zostaje wypełniony
- Możliwość edycji textarea/pliku i próby ponownie

US-023
Tytuł: Rate limiting dla AI generowania
Opis: Jako system chcę ograniczyć liczbę generacji AI na użytkownika, aby kontrolować koszty.
Kryteria akceptacji:
- Limit: 5 generacji na użytkownika na dzień (24h)
- Tracking w tabeli ai_generations (user_id, generated_at) lub cache (Redis)
- Przy próbie 6. generacji: błąd "Osiągnięto dzienny limit generacji (5). Spróbuj jutro lub stwórz quiz ręcznie."
- Licznik resetuje się po 24h od pierwszej generacji
- Wyświetlenie pozostałych generacji: "Pozostało: 3/5 generacji dzisiaj"

US-024
Tytuł: Walidacja rozmiaru uploadowanego pliku
Opis: Jako użytkownik uploadujący plik do AI chcę być poinformowany jeśli plik jest zbyt duży.
Kryteria akceptacji:
- Limity: PDF max 10MB, zdjęcia max 5MB
- Walidacja client-side (przed wysłaniem) przez JavaScript
- Jeśli plik > limit:
  - Przycisk "Generuj pytania" disabled
  - Błąd: "Plik jest zbyt duży. Maksymalny rozmiar: 10 MB (PDF) / 5 MB (zdjęcia)"
  - Input file highlighted czerwonym
- Użytkownik musi wybrać mniejszy plik
- Walidacja również server-side (backup)

US-025
Tytuł: Obsługa network error podczas gry
Opis: Jako gracz chcę aby moje odpowiedzi były zapisane nawet jeśli stracę połączenie internetowe.
Kryteria akceptacji:
- Odpowiedzi zapisywane w localStorage przed wysłaniem do serwera
- Przy network error: retry automatyczny (3 próby po 2s)
- Jeśli wszystkie fail: komunikat "Utracono połączenie. Sprawdź internet i odśwież stronę."
- Po odświeżeniu: próba recovery z localStorage
- Wysłanie zaległych odpowiedzi do serwera
- Kontynuacja gry od miejsca przerwania

US-026
Tytuł: Obsługa porzuconej sesji gry
Opis: Jako system chcę oznaczać sesje gry jako "porzucone" jeśli gracz nie ukończy quizu.
Kryteria akceptacji:
- Sesja ma status 'in_progress'
- Gracz zamyka kartę/przeglądarkę bez ukończenia
- Cron job (lub Supabase Function) uruchamiany co godzinę
- Jeśli game_session.started_at < NOW() - 1 hour AND status='in_progress':
  - Update status='abandoned'
- Porzucone sesje NIE pokazują się w leaderboard
- Statystyki dla twórcy pokazują liczbę porzuconych sesji

US-027
Tytuł: Weryfikacja własności quizu przy edycji
Opis: Jako system chcę weryfikować czy użytkownik jest właścicielem quizu przed umożliwieniem edycji.
Kryteria akceptacji:
- API route PUT /api/quizzes/:id sprawdza:
  - SELECT * FROM quizzes WHERE id=:id AND user_id=auth.uid()
- Jeśli quiz nie istnieje lub user_id != auth.uid(): HTTP 403 Forbidden
- Response: {"error": "Nie masz uprawnień do edycji tego quizu"}
- Frontend: przycisk "Edytuj" widoczny tylko dla własnych quizów
- Supabase RLS policy dodatkowe zabezpieczenie

### 5.7 Security i prywatność

US-028
Tytuł: Hashowanie haseł użytkowników
Opis: Jako system chcę hashować hasła użytkowników, aby chronić ich dane w przypadku wycieku bazy.
Kryteria akceptacji:
- Hasło nigdy nie jest zapisywane w plain text
- Supabase Auth automatycznie hashuje hasła (bcrypt)
- W tabeli auth.users przechowywany tylko password_hash
- Hasło nie jest zwracane w żadnym API response
- Walidacja hasła przez Supabase Auth (porównanie hash)

US-029
Tytuł: Row Level Security dla quizów
Opis: Jako system chcę zabezpieczyć dostęp do quizów na poziomie bazy danych, aby użytkownicy widzieli tylko swoje quizy.
Kryteria akceptacji:
- Supabase RLS policy dla tabeli quizzes:
  - SELECT: auth.uid() = user_id
  - INSERT: auth.uid() = user_id
  - UPDATE: auth.uid() = user_id
  - DELETE: auth.uid() = user_id
- Nawet jeśli ktoś zna quiz_id innego użytkownika, nie może go odczytać/edytować
- Pytania i odpowiedzi (questions, answers) są publiczne (dla graczy)
- Game_sessions są publiczne (dla leaderboard)

US-030
Tytuł: Sanityzacja inputów użytkownika
Opis: Jako system chcę sanityzować wszystkie inputy użytkownika, aby zapobiec atakom XSS i SQL injection.
Kryteria akceptacji:
- Frontend: React automatycznie escapuje content (nie używamy dangerouslySetInnerHTML)
- Backend: Supabase używa parametryzowanych queries (zapobiega SQL injection)
- Walidacja długości stringów (max lengths)
- Usuwanie HTML tags z user input (strip tags)
- Escape specjalnych znaków przed renderowaniem

US-031
Tytuł: HTTPS only dla aplikacji
Opis: Jako użytkownik chcę aby moja komunikacja z aplikacją była szyfrowana.
Kryteria akceptacji:
- Cała aplikacja dostępna tylko przez HTTPS
- Vercel/Netlify automatycznie zapewnia SSL certificate
- HTTP requests przekierowywane na HTTPS (301)
- Supabase API wymaga HTTPS
- Brak mixed content warnings

## 6. Metryki sukcesu

### 6.1 Kluczowe wskaźniki efektywności (KPIs)

Główne KPIs dla MVP (pomiar po 1 miesiącu od launch):

KPI-001: Adoption - Liczba zarejestrowanych użytkowników (twórców)
- Cel: Minimum 50 zarejestrowanych użytkowników
- Pomiar: COUNT(*) FROM users WHERE created_at >= launch_date
- Źródło: Baza danych Supabase + Analytics
- Success threshold: >= 50

KPI-002: Engagement - Procent użytkowników tworzących quizy
- Cel: 70% zarejestrowanych użytkowników tworzy przynajmniej 1 quiz
- Pomiar: (Liczba user_id w quizzes) / (Liczba users) * 100
- Źródło: Database query
- Success threshold: >= 70%

KPI-003: AI Usage - Procent quizów tworzonych z AI
- Cel: 50% quizów jest generowanych przez AI (nie ręcznie)
- Pomiar: Analytics event "AI generation success" / "Quiz created"
- Źródło: Analytics (Plausible/Posthog)
- Success threshold: >= 50%

KPI-004: Game Completion - Procent ukończonych gier
- Cel: 60% rozpoczętych gier jest ukończonych (nie porzuconych)
- Pomiar: (game_sessions WHERE status='completed') / (ALL game_sessions) * 100
- Źródło: Database query
- Success threshold: >= 60%

KPI-005: Virality - Średnia liczba graczy na quiz
- Cel: Każdy quiz jest rozgrywany średnio przez 5+ graczy
- Pomiar: AVG(COUNT game_sessions per quiz_id)
- Źródło: Database query
- Success threshold: >= 5 graczy/quiz

### 6.2 Metryki produktowe

Metryki tworzenia quizów:

METRIC-001: Liczba utworzonych quizów
- Całkowita liczba quizów w systemie
- Benchmark: >= 100 quizów w pierwszym miesiącu

METRIC-002: AI success rate
- Procent udanych generacji AI
- Cel: >= 90% sukces
- Pomiar: (AI success) / (AI attempts) * 100

METRIC-003: Średni czas generowania AI
- Średni czas odpowiedzi OpenRouter API
- Cel: < 30 sekund
- Benchmark: 15-25 sekund

METRIC-004: Średnia liczba pytań w quizie
- AVG(COUNT questions per quiz_id)
- Benchmark: 8-12 pytań

Metryki rozgrywki:

METRIC-005: Liczba rozegranych gier
- COUNT(*) FROM game_sessions WHERE status='completed'
- Benchmark: >= 500 gier w pierwszym miesiącu

METRIC-006: Średni wynik graczy
- AVG(total_points) z completed games
- Benchmark: 5000-7000 pkt (50-70% max)
- Wskazuje balans trudności pytań

METRIC-007: Średni czas rozgrywki
- AVG(completed_at - started_at) dla completed games
- Benchmark: 5-10 minut (zależnie od liczby pytań)

METRIC-008: Drop rate (porzucone gry)
- (status='abandoned') / (ALL games) * 100
- Cel: < 40%

METRIC-009: Repeat play rate
- Procent graczy którzy grają w ten sam quiz 2+ razy
- Pomiar: COUNT(player_nickname WHERE COUNT(game_sessions) >= 2) / COUNT(DISTINCT player_nickname)
- Cel: >= 20%

Metryki zaangażowania:

METRIC-010: Leaderboard views
- Liczba wyświetleń /quiz/:pin/leaderboard
- Benchmark: 80% graczy ogląda leaderboard po grze

METRIC-011: Share rate (przyszłość)
- Procent graczy którzy udostępniają quiz dalej
- Pomiar: tracking "Kopiuj PIN" clicks
- Cel: >= 30%

Metryki techniczne:

METRIC-012: API response time
- Średni czas odpowiedzi /api/* endpoints
- Cel: < 300ms dla 95% requestów
- Monitoring: Sentry Performance

METRIC-013: Error rate
- Procent requestów kończących się błędem
- Cel: < 2%
- Monitoring: Sentry

METRIC-014: Page load time
- Time to Interactive dla głównych stron
- Cel: < 2 sekundy
- Monitoring: Web Vitals

### 6.3 Metody pomiaru

Analytics:
- Plausible Analytics (privacy-first, GDPR compliant)
- Custom events dla kluczowych akcji
- Dashboard z real-time metrics

Database queries:
- Cotygodniowe SQL queries dla KPIs
- Supabase Dashboard + custom queries
- Export do CSV dla analizy

Error tracking:
- Sentry dla JavaScript errors i API errors
- Email alerts dla critical issues
- Dashboard z error rate trends

Performance monitoring:
- Sentry Performance lub Web Vitals
- Page load time, API response time
- Alerts gdy degradacja > 20%

User feedback:
- Opcjonalny feedback form po grze
- Email follow-up po tygodniu od rejestracji
- Zbieranie sugestii features

### 6.4 Kryteria sukcesu MVP

MVP zostanie uznany za sukces jeśli po 1 miesiącu:

✅ Osiągnięte zostaną KPIs:
- >= 50 zarejestrowanych użytkowników
- >= 70% użytkowników tworzy quizy
- >= 50% quizów z AI
- >= 60% gier ukończonych (nie porzuconych)
- >= 5 graczy/quiz średnio

✅ Stabilność techniczna:
- < 2% error rate
- AI success rate >= 90%
- API response time < 300ms

✅ Pozytywny feedback:
- Użytkownicy wyrażają chęć dalszego korzystania
- Brak krytycznych bugów
- Pozytywne komentarze w feedback

Jeśli sukces:
- Iteracja 2.0 z dodatkowymi features
- Marketing i growth strategies
- Rozważenie monetyzacji (freemium)

Jeśli nie:
- User interviews i analiza problemów
- Iteracja na podstawie feedbacku
- Pivot lub zmiany w product strategy

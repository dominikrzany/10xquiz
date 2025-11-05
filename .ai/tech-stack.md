# Stos technologiczny 10xQuiz

## Frontend - Astro z React dla komponentów interaktywnych

- **Astro 5** - pozwala na tworzenie szybkich, wydajnych stron i aplikacji z minimalną ilością JavaScript
- **React 19** - zapewni interaktywność tam, gdzie jest potrzebna
- **TypeScript 5** - dla statycznego typowania kodu i lepszego wsparcia IDE
- **Tailwind 4** - pozwala na wygodne stylowanie aplikacji
- **Shadcn/ui** - zapewnia bibliotekę dostępnych komponentów React, na których oprzemy UI

## Backend - Supabase jako kompleksowe rozwiązanie backendowe

- Zapewnia bazę danych **PostgreSQL**
- Zapewnia SDK w wielu językach, które posłużą jako Backend-as-a-Service
- Jest rozwiązaniem **open source**, które można hostować lokalnie lub na własnym serwerze
- Posiada wbudowaną **autentykację użytkowników**

## AI - OpenRouter do używania różnych modeli i komunikacji z AI

- Dostęp do **GPT-4** i innych modeli LLM
- Elastyczne przełączanie między modelami
- Proste API do generowania pytań quizowych

## CI/CD i Hosting

### GitHub Actions do tworzenia pipeline'ów CI/CD:

- **Pull Request Checks** - automatyczna walidacja PR (linting, unit tests, e2e tests z coverage)
- **Master Branch Validation** - testy i build po merge do master
- **Vercel Deployment** - automatyczny deploy do produkcji po pushu do master

### Vercel do hostowania aplikacji

- Serverless deployment platform
- Automatyczne skalowanie
- Edge network dla szybkiego ładowania na całym świecie
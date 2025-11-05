# 10xQuiz

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node Version](https://img.shields.io/badge/node-22.14.0-brightgreen.svg)](https://nodejs.org)
[![Built with Astro](https://img.shields.io/badge/Built%20with-Astro-ff5e00.svg)](https://astro.build)

An AI-powered interactive quiz platform that combines learning with gamification and entertainment. Create engaging quizzes from PDFs, images, links, or text using GPT-4, share them via PIN codes, and let players compete on the leaderboard.

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [Getting Started Locally](#getting-started-locally)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Environment Variables](#environment-variables)
  - [Running the Development Server](#running-the-development-server)
- [Available Scripts](#available-scripts)
- [Project Scope](#project-scope)
  - [What's Included in MVP](#whats-included-in-mvp)
  - [What's Not Included](#whats-not-included)
- [Project Structure](#project-structure)
- [Project Status](#project-status)
- [Contributing](#contributing)
- [License](#license)

## Overview

10xQuiz is a web application designed to revolutionize the way people create and play quizzes. Traditional learning and knowledge testing can be monotonous, and existing solutions are often expensive or complicated. 10xQuiz solves these problems by:

- **Automating quiz creation** with AI (GPT-4 via OpenRouter)
- **Simplifying sharing** with 6-digit PIN codes
- **Enabling asynchronous play** - everyone plays at their own pace
- **Motivating through competition** with real-time leaderboards and point systems (max 1000 points per question)
- **Removing barriers** - players don't need to register, just enter a nickname

### Target Users

- **Teachers and trainers** - need tools to quickly create quizzes from lesson materials
- **Students** - want to test their knowledge in a competitive way
- **Social groups** (friends, family) - looking for engaging entertainment
- **Anyone** who needs to create a quiz and motivate others to learn

## Key Features

### For Quiz Creators

- ✅ **User registration and authentication** (email/password via Supabase Auth)
- 🤖 **AI quiz generation** - Upload PDFs, images, or provide text/links to automatically generate questions
- ✏️ **Manual quiz creation** - Full control over questions and answers
- 📊 **Dashboard** - Manage all your quizzes in one place
- 🔢 **Automatic PIN generation** - Each quiz gets a unique 6-digit PIN
- 📈 **Analytics and leaderboard** - View detailed statistics and all player results
- 🎯 **Quiz editing** - Update questions and answers anytime

### For Players (No Registration Required!)

- 🎮 **PIN-based entry** - Just enter a PIN to start playing
- 👤 **Nickname only** - No account needed, just provide a nickname
- ⏱️ **Timed questions** - 5 seconds to read + 20 seconds to answer
- 💯 **Point system** - Earn up to 1000 points per question (faster = more points)
- 🏆 **Live leaderboard** - See how you rank against other players
- 🔄 **Replay option** - Try again to improve your score
- 📱 **Mobile-first design** - Optimized for smartphone gameplay

### Scoring System

```
Points = 1000 - (time_taken_ms / 20)
```

**Examples:**
- Answer in 100ms → 995 points
- Answer in 5000ms → 750 points
- Answer in 20000ms → 0 points
- Wrong answer → 0 points

## Tech Stack

### Frontend

- **[Astro 5](https://astro.build)** - Fast, modern web framework with minimal JavaScript
- **[React 19](https://react.dev)** - Interactive UI components
- **[TypeScript 5](https://www.typescriptlang.org)** - Static type checking
- **[Tailwind CSS 4](https://tailwindcss.com)** - Utility-first CSS framework
- **[Shadcn/ui](https://ui.shadcn.com)** - Accessible React component library

### Backend

- **[Supabase](https://supabase.com)** - Backend-as-a-Service
  - PostgreSQL database
  - Built-in authentication
  - Row Level Security (RLS)
  - Real-time capabilities

### AI

- **[OpenRouter](https://openrouter.ai)** - Access to GPT-4 and other LLMs
  - AI-powered question generation from various content formats
  - Flexible model switching

### CI/CD & Hosting

- **GitHub Actions** - Automated testing and deployment
  - Pull request validation (linting, tests)
  - Master branch checks
  - Automatic deployment
- **[Vercel](https://vercel.com)** - Serverless deployment platform
  - Automatic scaling
  - Global edge network
  - Preview deployments

## Getting Started Locally

### Prerequisites

- **Node.js** `22.14.0` (use [nvm](https://github.com/nvm-sh/nvm) for version management)
- **npm** or **pnpm** package manager
- **Supabase account** (for database and authentication)
- **OpenRouter API key** (for AI quiz generation)

### Installation

1. **Clone the repository**

```bash
git clone https://github.com/dominikrzany/10xquiz.git
cd 10xquiz
```

2. **Install Node.js version**

```bash
nvm use
```

This will automatically use the version specified in `.nvmrc` (22.14.0)

3. **Install dependencies**

```bash
npm install
```

### Environment Variables

Create a `.env` file in the root directory with the following variables:

```env
# Supabase
PUBLIC_SUPABASE_URL=your_supabase_project_url
PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key

# OpenRouter AI
OPENROUTER_API_KEY=your_openrouter_api_key

# Application
PUBLIC_APP_URL=http://localhost:3000
```

**Setting up Supabase:**

1. Create a project at [supabase.com](https://supabase.com)
2. Find your API keys in Project Settings → API
3. Run the database migrations (see `src/db/schema.sql`)
4. Enable Row Level Security policies

**Getting OpenRouter API Key:**

1. Sign up at [openrouter.ai](https://openrouter.ai)
2. Generate an API key from your dashboard
3. Add credits to your account for API usage

### Running the Development Server

```bash
npm run dev
```

The application will be available at `http://localhost:3000`

## Available Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start development server on `localhost:3000` |
| `npm run build` | Build the production site to `./dist/` |
| `npm run preview` | Preview the built site locally before deployment |
| `npm run astro` | Run Astro CLI commands |
| `npm run lint` | Check code for linting errors |
| `npm run lint:fix` | Automatically fix linting errors |
| `npm run format` | Format code with Prettier |

## Project Scope

### What's Included in MVP

✅ **Core Features:**
- User registration and authentication (creators only)
- AI-powered quiz generation from PDFs, images, text, and links
- Manual quiz creation and editing
- 6-digit PIN-based quiz sharing
- Complete gameplay flow with timed questions
- Point calculation based on speed and correctness
- Real-time leaderboards (TOP 10 + player rank)
- Quiz analytics for creators (stats, all results, filtering)
- Mobile-first responsive design
- Dashboard for quiz management

✅ **Technical Features:**
- PostgreSQL database with Row Level Security
- Rate limiting for AI generation (5/day per user)
- Session management for async gameplay
- Error handling and validation
- Abandoned session tracking

### What's Not Included

The following features are **NOT** included in the MVP but may be considered for future iterations:

❌ **Advanced User Features:**
- Social login (Google, Facebook, Apple)
- Password reset / forgot password
- Email verification
- User profiles with avatars
- Player registration (players remain anonymous)

❌ **Advanced Quiz Features:**
- Public quiz catalog
- Quiz ratings and comments
- Quiz duplication
- Import/export (JSON, CSV)
- Categories and tags
- Questions with images/videos
- Multiple question types (true/false, multi-select, open-ended)

❌ **Advanced Gameplay:**
- Real-time multiplayer mode
- Team mode
- Power-ups or bonuses
- Pause functionality during quiz
- Overall quiz time limit

❌ **Advanced Analytics:**
- Detailed question-level statistics
- Export to PDF/CSV
- Charts and visualizations
- Individual player progress tracking
- A/B testing

❌ **Integrations & Social:**
- LMS integration (Google Classroom, Moodle)
- External knowledge sources (Wikipedia, YouTube)
- Public API for developers
- Social media sharing
- Chat during quiz

❌ **Other Features:**
- Multi-language support (i18n)
- Dark mode
- Offline mode
- Native mobile apps
- White-label for organizations
- Monetization (freemium, ads)

## Project Structure

```
10xquiz/
├── .ai/                    # AI rules and project documentation
├── public/                 # Static assets (favicon, images)
├── src/
│   ├── assets/            # Internal static assets
│   ├── components/        # React and Astro components
│   │   └── ui/           # Shadcn/ui components
│   ├── db/               # Supabase clients and types
│   ├── layouts/          # Astro layouts
│   ├── lib/              # Services and helper functions
│   ├── middleware/       # Astro middleware (auth, etc.)
│   ├── pages/            # Astro pages (routes)
│   │   └── api/         # API endpoints
│   ├── styles/           # Global CSS styles
│   ├── types.ts          # Shared TypeScript types
│   └── env.d.ts          # Environment type definitions
├── astro.config.mjs      # Astro configuration
├── components.json       # Shadcn/ui configuration
├── eslint.config.js      # ESLint configuration
├── package.json          # Dependencies and scripts
├── tsconfig.json         # TypeScript configuration
└── README.md            # This file
```

## Project Status

🚧 **Current Status:** In Development (MVP Phase)

### Roadmap

- [x] Project setup and tech stack configuration
- [ ] Database schema and Supabase setup
- [ ] User authentication (register/login)
- [ ] Dashboard and quiz management
- [ ] AI quiz generation integration
- [ ] Quiz gameplay flow
- [ ] Leaderboard and results
- [ ] Analytics for quiz creators
- [ ] Mobile optimization
- [ ] Testing and bug fixes
- [ ] Production deployment

### Success Metrics (After 1 Month)

The MVP will be considered successful if:

- ✅ **50+ registered users** (quiz creators)
- ✅ **70% of users create at least 1 quiz**
- ✅ **50% of quizzes are AI-generated**
- ✅ **60% of started games are completed** (not abandoned)
- ✅ **5+ players per quiz** on average
- ✅ **<2% error rate** and **90%+ AI success rate**
- ✅ **Positive user feedback** and willingness to continue using

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Code Quality

- Run `npm run lint` before committing
- Ensure all tests pass
- Follow the existing code style
- Add tests for new features

## License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

---

**Built with ❤️ using Astro, React, and AI**

For questions or support, please open an issue on GitHub.

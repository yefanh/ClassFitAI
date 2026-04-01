# EduConnect

EduConnect is a Hackathon project designed for course selection support:

- The frontend collects student learning preferences via a survey (lecture attendance, workload tolerance, exam confidence, question-asking habits, etc.).
- The backend gathers instructor data and review content for a given course.
- An LLM converts text reviews into structured rating dimensions.
- The system computes professor compatibility scores based on user preferences and returns recommendations.

## Project Structure

```text
educonnect-master/
  backend/
    main.py          # Flask API entrypoint
    login.py         # Uses Selenium to fetch instructors from Northeastern Banner
    ratemyprof.py    # Fetches RateMyProfessors comments
    llm.py           # OpenAI calls and review structuring
    Algorithm.py     # Compatibility scoring logic
    Test.csv         # Sample test data
  frontend/
    index.html       # Static login page
    landing.html     # Feature selection page
    profrecs.html    # Professor recommendation survey page
    class_schedule.html
    profscripts.js   # Frontend interactions and API calls
    style.css / styler.css / profstyles.css
    vercel.json
```

## Tech Stack

- Frontend: HTML + CSS + JavaScript (static pages)
- Backend: Python + Flask
- Data scraping: Selenium + requests + BeautifulSoup
- AI: OpenAI ChatCompletion (currently configured with `gpt-3.5-turbo` in code)

## Run Locally

### 1. Prepare Python Environment (3.9+ recommended)

Install dependencies under `backend/`:

```bash
cd backend
pip install flask selenium selenium-wire webdriver-manager requests beautifulsoup4 openai
```

### 2. Configure OpenAI Key

`backend/llm.py` reads an `openai.key` file from the same directory.

Create this file under `backend/`:

```text
openai.key
```

Put only your API key in the file (no quotes, no extra spaces).

### 3. Start the Backend

```bash
cd backend
python main.py
```

Default endpoint:

- `http://localhost:5000`
- Recommendation API: `POST /submit`

### 4. Start the Frontend

The frontend is static, so you can serve `frontend/` with any static file server.

Example:

```bash
cd frontend
python -m http.server 5500
```

Then open:

- `http://localhost:5500/index.html`

The frontend sends requests to `http://localhost:5000/submit`, so the backend must run at the same time.

## API

### `POST /submit`

Request body example:

```json
{
  "class_code": "cs2510",
  "ratings": {
    "lecture_frequency": "7",
    "lecture_importance": "8",
    "exam_ability": "6",
    "in_class_question_frequency": "5",
    "in_class_question_importance": "6",
    "forum_question_frequency": "4",
    "forum_question_weight": "7",
    "workload_handling_ability": "6"
  }
}
```

Response example (key = professor name, value = compatibility score):

```json
{
  "First Last": 82.4,
  "Another Prof": 76.8
}
```

## Current Processing Flow

1. The frontend submits survey answers and a course code.
2. `login.py` uses Selenium to fetch instructors from the school registration system.
3. `ratemyprof.py` fetches corresponding RateMyProfessors comments.
4. `llm.py` converts comments into 0-100 scores across several dimensions and adds strengths/weaknesses text.
5. `Algorithm.py` computes compatibility based on user preferences.
6. The backend returns results to the frontend for dynamic display.

## Known Caveats

- `login.py` currently hardcodes the term as `Spring 2024 Semester`; update it to the active term.
- Selenium scraping depends on a browser runtime; headless/CI environments may need extra setup.
- `llm.py` uses an older OpenAI SDK pattern (`openai.ChatCompletion.create`); migration may be needed when upgrading.
- `frontend/contact.html` is currently empty.
- `frontend/portfolio.html` references `styles.css`, while the repo primarily uses `style.css`; adjust if you plan to use that page.

## Quick Troubleshooting

- No recommendation result in frontend:
  - Confirm backend is running on `localhost:5000`.
  - Check browser console for `fetch` errors.
- OpenAI-related backend errors:
  - Ensure `backend/openai.key` exists and contains a valid key.
- Selenium errors:
  - Ensure a local browser is available and target websites are reachable.

## Potential Improvements

- Add `requirements.txt` and one-command startup scripts.
- Move secrets to `.env` instead of plaintext key files.
- Improve backend error handling and structured logging.
- Make frontend/backend API base URL configurable.
- Add unit tests and API integration tests.

---

If you want, I can implement these next:

1. `requirements.txt`
2. `.env` + config loading
3. Developer startup script + docs

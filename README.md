# IncidentIQ

**An AI-powered incident-response assistant that helps engineering teams learn from past incidents.**

IncidentIQ uses long-term memory and AI-generated troubleshooting guidance to help engineers investigate service failures with historical context, rather than starting from scratch every time.

## The Problem

Engineering teams often encounter recurring incidents, but useful knowledge is scattered across old incident reports and troubleshooting notes. This can make investigations slower and lead to repeated work.

## Our Solution

IncidentIQ connects a new incident with relevant historical incidents and generates context-aware troubleshooting suggestions. Engineers review the suggestions, perform diagnostic checks, verify the resolution, and save the outcome for future investigations.

## How It Works

1. **Report an incident:** Enter the symptoms and relevant context in the dashboard.
2. **Retrieve memories:** Hindsight retrieves relevant historical incident records.
3. **Generate suggestions:** Groq runs GPT-OSS 20B to generate a troubleshooting plan using the current incident and retrieved context.
4. **Investigate and verify:** The engineer reviews the suggestions and performs appropriate checks.
5. **Save the outcome:** Record the resolution, outcome, and verification status in Hindsight for future use.

## Tech Stack

- **Python** – Backend logic
- **FastAPI** – API and service endpoints
- **Hindsight** – Long-term memory and incident retrieval
- **Groq API** – AI inference
- **GPT-OSS 20B** – Language model for troubleshooting suggestions
- **HTML, CSS, JavaScript** – Frontend dashboard
- **Pydantic** – Request validation
- **Uvicorn** – Local ASGI server

## Features

- Store and retrieve historical incident records
- Generate AI-assisted troubleshooting suggestions
- Use previous incident context during investigation
- Record resolutions and verification status
- Keep engineers in control of production decisions

## Getting Started

### Prerequisites

- Python 3.10 or a compatible version supported by your dependencies
- Git
- A Groq API key
- A running Hindsight API service

### Installation

1. Clone this repository:

   ```bash
   git clone https://github.com/YOUR-USERNAME/incidentiq.git
   cd incidentiq
   ```

2. Create and activate a virtual environment on Windows:

   ```powershell
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1
   ```

3. Install dependencies:

   ```powershell
   pip install -r requirements.txt
   ```

4. Create a `.env` file in the project root and add your Groq API key using the variable name expected by `app.py`:

   ```text
   GROQ_API_KEY=your_groq_api_key_here
   ```

   Never commit your `.env` file or share your API key.

5. Start the Hindsight API using its official setup instructions and ensure it is available at the URL configured in `app.py`.

6. Start the FastAPI backend:

   ```powershell
   python -m uvicorn app:app --port 8000
   ```

7. Open `index.html` in your browser. Keep the backend and Hindsight services running while using the app.

## API Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/health` | Check backend health |
| POST | `/api/recall` | Retrieve relevant incident memories |
| POST | `/api/suggest` | Generate troubleshooting suggestions |
| POST | `/api/retain` | Save an incident record |

## Safety and Human Oversight

IncidentIQ provides decision support, not autonomous remediation. Engineers should review AI-generated suggestions, verify findings, and authorize any production changes themselves. AI-generated suggestions may be incorrect and must be checked against actual evidence.

## Project Status

Developed as a hackathon project. Features and setup instructions may evolve as development continues.

## Author

Built for Hack With Hyderabad 3.0.

## License

No license has been specified yet. Add a license if you intend to define how others may use, modify, and distribute this project.
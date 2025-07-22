# GitRead

[![Ask DeepWiki](https://devin.ai/assets/askdeepwiki.png)](https://deepwiki.com/gitreadhq/gitread)

GitRead is an AI-powered documentation generator that transforms any public GitHub repository into a professional, ready-to-use README file. By analyzing the repository's structure, code, and metadata, GitRead crafts comprehensive and well-structured documentation with minimal effort.

## Key Features

*   **AI-Powered Generation**: Leverages large language models (Gemini Pro via OpenRouter) to understand your code and generate high-quality, human-like documentation.
*   **Public Repository Support**: Simply provide the URL of any public GitHub repository to get started.
*   **Markdown & Preview**: View the generated README in both raw Markdown and a rendered HTML preview.
*   **Edit, Copy, and Download**: Easily edit the generated content, copy it to your clipboard, or download it as a `README.md` file.
*   **Generation History**: Authenticated users can access a history of their previously generated READMEs.
*   **Credit-Based System**: Users receive an initial credit to try the service and can purchase more via Stripe for continued use.
*   **Secure Authentication**: User accounts and authentication are managed securely using Clerk.

## How It Works

GitRead follows a simple yet powerful process to generate documentation:

1.  **URL Submission**: The user provides a public GitHub repository URL.
2.  **Repository Ingestion**: A separate Python microservice clones the repository, analyzes its structure, and extracts relevant content from source files. This process respects size and token limits to ensure efficient processing.
3.  **AI Prompting**: The extracted context, including a file tree and code snippets, is formatted into a detailed prompt.
4.  **README Generation**: The prompt is sent to the Gemini Pro AI model via OpenRouter, which generates the complete README file.
5.  **Display and Refine**: The generated README is displayed to the user in a feature-rich editor, allowing for final adjustments, copying, or downloading.

## Tech Stack

GitRead is built with a modern, robust technology stack:

*   **Framework**: [Next.js](https://nextjs.org/) (React)
*   **Styling**: [Tailwind CSS](https://tailwindcss.com/)
*   **Authentication**: [Clerk](https://clerk.com/)
*   **Database**: [Supabase](https://supabase.io/) (PostgreSQL)
*   **Payments**: [Stripe](https://stripe.com/)
*   **AI Model**: Google Gemini Pro via [OpenRouter](https://openrouter.ai/)
*   **Repository Ingestion Service**: A custom Python microservice using the `gitingest` library, deployed on [Render](https://render.com/).

## Architecture Overview

The system is designed as a Next.js monolith for the frontend and primary backend logic, which communicates with a dedicated Python microservice for the heavy lifting of repository analysis.

1.  **Next.js Application (Frontend & Backend)**:
    *   The frontend is built with React and provides the user interface for entering repository URLs, viewing generated content, and managing user accounts.
    *   Next.js API Routes serve as the backend, handling user authentication, managing database operations with Supabase, processing Stripe payments, and orchestrating the README generation workflow.

2.  **Python Ingestion Service**:
    *   A lightweight, standalone service responsible for cloning and parsing GitHub repositories.
    *   It takes a repository URL as input and returns a structured JSON object containing the repository's file tree, a summary, and the concatenated content of relevant files.

3.  **External Services**:
    *   **Clerk**: Manages user sign-up, sign-in, and session management.
    *   **Supabase**: Provides the database for storing user credits, generated README history, and processed Stripe payment events.
    *   **Stripe**: Handles all credit card processing for purchasing new credits.
    *   **OpenRouter**: Acts as a gateway to the Gemini Pro LLM, which generates the final README content.

## Local Development Setup

To run GitRead locally, follow these steps.

### Prerequisites

*   Node.js (v18 or later)
*   Python 3.8+
*   An active account with [Supabase](https://supabase.io/), [Clerk](https://clerk.com/), [Stripe](https://stripe.com/), and [OpenRouter](https://openrouter.ai/).

### Installation

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/gitreadhq/gitread.git
    cd gitread
    ```

2.  **Install frontend dependencies:**
    ```bash
    npm install
    ```

3.  **Set up environment variables:**
    Create a `.env.local` file in the root of the project and add the following keys. Obtain the values from your respective service provider dashboards.

    ```env
    # Clerk
    NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
    CLERK_SECRET_KEY=

    # Supabase
    NEXT_PUBLIC_SUPABASE_URL=
    NEXT_PUBLIC_SUPABASE_ANON_KEY=
    SUPABASE_SERVICE_ROLE_KEY=

    # OpenRouter
    OPENROUTER_API_KEY=

    # Stripe
    STRIPE_SECRET_KEY=

    # Application URL
    NEXT_PUBLIC_APP_URL=http://localhost:3000

    # Python Ingestion Service
    PYTHON_API_KEY= # A secret key to secure your Python API
    ```

4.  **Set up the Database:**
    *   Navigate to the Supabase SQL Editor.
    *   Execute the contents of the migration files located in `supabase/migrations/` to create the required tables (`user_credits`, `generated_readmes`, `processed_stripe_events`) and set up Row Level Security (RLS) policies.

5.  **Run the development server:**
    ```bash
    npm run dev
    ```
    The application will be available at `http://localhost:3000`.

**Note:** The Python ingestion service (`scripts/git_ingest.py`) is designed to be run as a separate microservice (e.g., on a platform like Render). For local development, you will need to set up a local server (e.g., using Flask or FastAPI) to run this script and expose it as an API endpoint.

## API Endpoints

The core backend functionality is exposed through several API routes:

*   `POST /api/generate`: The main endpoint that orchestrates the README generation. It handles authentication, checks user credits, calls the Python ingestion service, and then calls the AI model.
*   `GET /api/credits`: Fetches the current user's credit balance.
*   `POST /api/credits`: Updates the user's credit balance (used internally by admin functions).
*   `GET /api/readme-history`: Retrieves the list of previously generated READMEs for the authenticated user.
*   `POST /api/readme-history`: Saves a newly generated README to the user's history.
*   `POST /api/create-checkout-session`: Creates a Stripe checkout session for purchasing credits.
*   `POST /api/verify-payment`: Verifies a successful payment with Stripe and updates the user's credit balance accordingly.

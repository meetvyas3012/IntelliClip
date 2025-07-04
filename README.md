# IntelliClip

## Overview

This project is an AI-powered SaaS (Software as a Service) application designed to automatically generate short-form video clips from full-length podcast episodes. Leveraging cutting-edge AI models and serverless infrastructure, it streamlines the content creation process for platforms like YouTube Shorts and TikTok, helping creators repurpose long-form content efficiently.

## Features

* **Intelligent Clip Generation:** Automatically identifies "viral moments" (questions, stories, key discussions) from podcast transcripts using a Large Language Model (LLM).
* **Accurate Transcription:** Utilizes an advanced AI model (WhisperX) for precise transcription of audio, including word-level timestamps.
* **Active Speaker Detection:** Employs an AI model (LR-ASD) to automatically crop video clips, focusing on the active speaker's face for optimized vertical video formats.
* **GPU-Accelerated Video Rendering:** Fast and efficient video processing and rendering using serverless GPUs (NVIDIA L40S on Modal) with FFMPEGCV.
* **Automated Subtitle Generation:** Creates dynamic, styled subtitles for clips, enhancing accessibility and engagement.
* **Scalable Backend:** Built with Modal and Inngest for robust background processing, queue management, and serverless GPU execution.
* **Secure File Storage:** Integrates with AWS S3 for secure and scalable storage of raw and processed video files.
* **User Authentication:** Secure user management for a multi-tenant SaaS experience.
* **Credit-Based System & Payments:** Implements a credit-based system for clip generation, integrated with Stripe for seamless credit pack purchases.
* **Modern Frontend:** A responsive and intuitive web interface built with Next.js and Tailwind CSS.
* **Real-time Status Updates:** Dashboard to monitor uploaded podcasts, processing status, and generated clips.

## Technologies Used

**Frontend:**
* **Next.js:** React framework for building server-rendered and static web applications.
* **React:** JavaScript library for building user interfaces.
* **TypeScript:** Superset of JavaScript for type-safe code.
* **Tailwind CSS:** Utility-first CSS framework for rapid UI development.
* **Shadcn UI:** Reusable UI components built with Tailwind CSS and Radix UI.
* **Sonner:** Toast notifications.
* **Prisma:** Next-generation ORM for database interactions.

**Backend (AI Processing & Orchestration):**
* **Python:** Primary language for AI models and processing logic.
* **Modal:** Serverless platform for deploying and scaling Python code, especially with GPUs.
* **Inngest:** Event-driven serverless platform for background jobs, queues, and workflow orchestration.
* **FastAPI:** Python web framework for building API endpoints.
* **WhisperX:** AI model for robust audio transcription and alignment.
* **LR-ASD (Active Speaker Detection):** Custom or adapted model for identifying active speakers.
* **FFMPEGCV:** GPU-accelerated video processing library.
* **Google Gemini API:** Large Language Model for viral moment identification.
* **Boto3:** AWS SDK for Python (for S3 interaction).
* **Pysubs2:** Library for subtitle manipulation.

**Infrastructure & Services:**
* **AWS S3:** Object storage for media files.
* **Stripe:** Payment processing for credit purchases.
* **PostgreSQL (e.g., Neon DB):** Relational database for application data (users, files, clips, credits).
* **Git:** Version control.
* **Vercel:** Platform for deploying the Next.js frontend.

## Getting Started

Follow these steps to set up and run the project locally.

### Prerequisites

* Node.js (v18 or higher)
* Python (v3.11 or higher)
* Git
* AWS Account (with S3 bucket and IAM user credentials)
* Stripe Account (for API keys and webhooks)
* Modal Labs Account
* Inngest Account
* Google Cloud Account (for Gemini API key)
* PostgreSQL Database (e.g., a free tier from Neon.tech, Supabase, or a local Docker instance)

### 1. Clone the Repository

```bash
git clone [https://github.com/meetvyas3012/IntelliClip.git
````

### 2\. AWS S3 Setup

1.  **Create an S3 Bucket:** Create a new S3 bucket (e.g., `ai-podcast-clipper-1`) in your AWS console.
2.  **Configure CORS:** Add a CORS configuration to your S3 bucket to allow uploads from your frontend.
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <CORSConfiguration xmlns="[http://s3.amazonaws.com/doc/2006-03-01/](http://s3.amazonaws.com/doc/2006-03-01/)">
    <CORSRule>
        <AllowedOrigin>*</AllowedOrigin>
        <AllowedMethod>GET</AllowedMethod>
        <AllowedMethod>PUT</AllowedMethod>
        <AllowedMethod>POST</AllowedMethod>
        <AllowedHeader>*</AllowedHeader>
    </CORSRule>
    </CORSConfiguration>
    ```
3.  **Create an IAM User:** Create an IAM user with programmatic access. Grant this user `s3:GetObject`, `s3:PutObject`, and `s3:ListBucket` permissions on your S3 bucket.
4.  **Obtain Credentials:** Note down the `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.

### 3\. Database Setup (PostgreSQL & Prisma)

1.  **Create a PostgreSQL Database:**
      * **Recommended (Neon.tech):** Sign up for Neon.tech, create a new project, and copy your `DATABASE_URL` (connection string).
      * **Other:** Use Supabase, Railway, or set up a local PostgreSQL instance.
2.  **Configure `DATABASE_URL`:** Create a `.env` file in the `frontend` directory and add your database URL:
    ```env
    # frontend/.env
    DATABASE_URL="postgresql://YOUR_USER:YOUR_PASSWORD@YOUR_HOST:YOUR_PORT/YOUR_DATABASE?schema=public"
    ```
3.  **Run Prisma Migrations:**
    Navigate to the `frontend` directory and apply your Prisma schema to the database.
    ```bash
    cd frontend
    npm install # or yarn install / pnpm install
    npx prisma migrate dev --name init
    ```
    This will create the `User`, `Account`, `Session`, `UploadedFile`, `Clip`, `Post` (if applicable) and other tables in your database.

### 4\. Modal Backend Setup

1.  **Install Modal CLI:**
    ```bash
    pip install modal-client
    ```
2.  **Authenticate Modal:**
    ```bash
    modal token new
    ```
    Follow the browser prompts to authenticate.
3.  **Configure Modal Secrets:** Create a Modal Secret to securely store your AWS and Google Gemini API keys.
    ```bash
    modal secret create ai-youtube-clipper \
        AWS_ACCESS_KEY_ID="YOUR_AWS_ACCESS_KEY_ID" \
        AWS_SECRET_ACCESS_KEY="YOUR_AWS_SECRET_ACCESS_KEY" \
        AWS_DEFAULT_REGION="YOUR_AWS_REGION" \
        GEMINI_API_KEY="YOUR_GEMINI_API_KEY" \
        AUTH_TOKEN="YOUR_CUSTOM_AUTH_TOKEN" # A token for your frontend to authenticate with Modal
    ```
    Replace placeholders with your actual keys and region. `AUTH_TOKEN` is a custom token you define for your frontend to send to your Modal endpoint for basic authentication.
4.  **Deploy Modal App:**
    Navigate to your `backend` directory and deploy your Modal application.
    ```bash
    cd ../backend # Assuming you are in frontend directory
    pip install -r requirements.txt # Install Python dependencies for backend
    modal deploy main.py::app
    ```
    Note the deployed URL for your `process_video` endpoint (e.g., `https://your-org-name--ai-podcast-clipper-aipodcastclipper-pr-xxxxxx-dev.modal.run/`).

### 5\. Inngest Setup

1.  **Get Inngest API Key:** Sign up for Inngest and get your Inngest API key.
2.  **Configure Inngest Environment Variable:** Add the Inngest key to your `frontend/.env` file.
    ```env
    # frontend/.env
    INNGEST_EVENT_KEY="YOUR_INNGEST_EVENT_KEY"
    INNGEST_SIGNING_KEY="YOUR_INNGEST_SIGNING_KEY" # From Inngest dashboard
    ```
3.  **Configure Modal Endpoint in Inngest Function:** Your Inngest function (defined in your frontend or a separate Inngest backend) will need to know the Modal endpoint URL. You can pass this as an environment variable to your Inngest function's deployment.

### 6\. Stripe Setup

1.  **Get Stripe API Keys:** Obtain your Stripe `STRIPE_SECRET_KEY` and `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` from your Stripe Dashboard.
2.  **Configure Stripe Environment Variables:** Add them to your `frontend/.env` file.
    ```env
    # frontend/.env
    STRIPE_SECRET_KEY="sk_test_..."
    NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY="pk_test_..."
    NEXT_PUBLIC_STRIPE_WEBHOOK_SECRET="whsec_..." # For webhook verification
    ```
3.  **Configure Stripe Webhooks:** Set up a webhook in Stripe to listen for events like `checkout.session.completed` and `invoice.payment_succeeded`. Point the webhook URL to an API route in your Next.js application (e.g., `/api/webhooks/stripe`).

### 7\. Frontend Environment Variables

Ensure your `frontend/.env` file contains all necessary environment variables:

```env
# frontend/.env

# Database
DATABASE_URL="postgresql://YOUR_USER:YOUR_PASSWORD@YOUR_HOST:YOUR_PORT/YOUR_DATABASE?schema=public"

# NextAuth.js (for authentication)
NEXTAUTH_SECRET="YOUR_NEXTAUTH_SECRET" # Generate a strong random string
NEXTAUTH_URL="http://localhost:3000" # For local development

# Inngest
INNGEST_EVENT_KEY="YOUR_INNGEST_EVENT_KEY"
INNGEST_SIGNING_KEY="YOUR_INNGEST_SIGNING_KEY"

# Stripe
STRIPE_SECRET_KEY="sk_test_..."
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY="pk_test_..."
NEXT_PUBLIC_STRIPE_WEBHOOK_SECRET="whsec_..."

# Modal (if frontend directly calls it, otherwise Inngest handles it)
NEXT_PUBLIC_MODAL_API_URL="[https://your-org-name--ai-podcast-clipper-aipodcastclipper-pr-xxxxxx-dev.modal.run/](https://your-org-name--ai-podcast-clipper-aipodcastclipper-pr-xxxxxx-dev.modal.run/)" # Base URL for your Modal app if needed by client
MODAL_VIDEO_PROCESSING_ENDPOINT="[https://your-org-name--ai-podcast-clipper-aipodcastclipper-pr-xxxxxx-dev.modal.run/process-video](https://your-org-name--ai-podcast-clipper-aipodcastclipper-pr-xxxxxx-dev.modal.run/process-video)" # Specific endpoint for video processing
```

Remember to replace all placeholder values with your actual keys and URLs. For `NEXTAUTH_SECRET`, you can generate one using `openssl rand -base64 32`.

## Running the Application

### Local Development

1.  **Start Frontend:**
    Navigate to the `frontend` directory and run the development server:

    ```bash
    cd frontend
    npm run dev
    ```

    Your frontend will be accessible at `http://localhost:3000`.

2.  **Test Modal Endpoint (Optional, for debugging):**
    You can test your Modal endpoint locally using the `main` entrypoint in your `backend/main.py` file:

    ```bash
    cd backend
    modal run main.py
    ```

    This will trigger a test request to your deployed Modal endpoint.

### Deployment

  * **Frontend (Next.js):** Deploy your `frontend` directory to Vercel. Ensure all environment variables are configured in Vercel's project settings.
  * **Backend (Modal):** Your Modal application is already deployed in the previous setup steps. Modal handles its scaling and execution.
  * **Inngest:** Your Inngest functions are typically deployed as part of your Next.js application (via API routes) or as separate serverless functions. Ensure Inngest environment variables are set in your deployment platform.

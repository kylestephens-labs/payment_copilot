# Getting Started

This guide will help you set up and run the Payment Copilot application locally.

## Prerequisites

Before you begin, ensure you have the following installed:

- [Node.js](https://nodejs.org/) (version 18 or higher)
- [npm](https://www.npmjs.com/) or [yarn](https://yarnpkg.com/)
- [Git](https://git-scm.com/)

## Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd payment_copilot
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

## Environment Setup

Create a `.env.local` file in the root directory with the following variables:

```env
# Firebase Configuration
NEXT_PUBLIC_FIREBASE_API_KEY=your_api_key
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
NEXT_PUBLIC_FIREBASE_PROJECT_ID=your_project_id
NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your_project.appspot.com
NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
NEXT_PUBLIC_FIREBASE_APP_ID=your_app_id

# AI Service API Keys
OPENAI_API_KEY=your_openai_api_key
ANTHROPIC_API_KEY=your_anthropic_api_key
REPLICATE_API_TOKEN=your_replicate_token
DEEPGRAM_API_KEY=your_deepgram_api_key
```

### Getting API Keys

#### Firebase
1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Create a new project or select existing one
3. Go to Project Settings > General
4. Scroll down to "Your apps" and add a web app
5. Copy the configuration values

#### OpenAI
1. Visit [OpenAI API](https://platform.openai.com/api-keys)
2. Create a new API key
3. Copy the key to your `.env.local` file

#### Anthropic
1. Visit [Anthropic Console](https://console.anthropic.com/)
2. Create a new API key
3. Copy the key to your `.env.local` file

#### Replicate
1. Visit [Replicate](https://replicate.com/)
2. Go to Account Settings > API Tokens
3. Create a new token
4. Copy the token to your `.env.local` file

#### Deepgram
1. Visit [Deepgram Console](https://console.deepgram.com/)
2. Create a new API key
3. Copy the key to your `.env.local` file

## Running the Application

1. **Start the development server**
   ```bash
   npm run dev
   ```

2. **Open your browser**
   Navigate to [http://localhost:3000](http://localhost:3000)

## Project Structure

```
src/
├── app/                    # Next.js App Router pages
│   ├── api/               # API routes
│   │   ├── anthropic/     # Anthropic API endpoints
│   │   ├── deepgram/      # Deepgram API endpoints
│   │   ├── openai/        # OpenAI API endpoints
│   │   └── replicate/     # Replicate API endpoints
│   ├── globals.css        # Global styles
│   ├── layout.tsx         # Root layout
│   └── page.tsx           # Home page
├── components/            # React components
│   ├── ImageUpload.tsx
│   ├── SignInWithGoogle.tsx
│   └── VoiceRecorder.tsx
└── lib/                   # Utility libraries
    ├── contexts/          # React contexts
    ├── firebase/          # Firebase configuration
    └── hooks/             # Custom React hooks
```

## Available Scripts

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint

## Troubleshooting

### Common Issues

1. **Environment variables not loading**
   - Ensure your `.env.local` file is in the root directory
   - Restart the development server after adding new variables
   - Check that variable names start with `NEXT_PUBLIC_` for client-side access

2. **Firebase authentication not working**
   - Verify your Firebase configuration
   - Check that authentication is enabled in Firebase Console
   - Ensure Google Sign-In is configured

3. **API calls failing**
   - Verify your API keys are correct
   - Check the browser console for error messages
   - Ensure you have sufficient API credits

## Next Steps

- Read the [API Reference](./api/README.md) to understand available endpoints
- Check out [Components](./components/README.md) for UI components
- Learn about [Authentication](./authentication.md) setup

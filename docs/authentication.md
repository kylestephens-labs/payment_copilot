# Authentication

This document covers the authentication system implemented in Payment Copilot using Firebase Authentication.

## Overview

Payment Copilot uses Firebase Authentication for user management and authentication. The system supports Google Sign-In and can be extended to support additional providers.

## Architecture

### Authentication Context

The authentication state is managed through React Context (`AuthContext`) located at `src/lib/contexts/AuthContext.tsx`.

**Key Features:**
- Real-time authentication state updates
- Automatic token refresh
- User session persistence
- Error handling

### Authentication Hook

A custom hook `useAuth` is available at `src/lib/hooks/useAuth.ts` for easy access to authentication state and methods.

**Usage:**
```tsx
import { useAuth } from '@/lib/hooks/useAuth';

function MyComponent() {
  const { user, loading, signIn, signOut } = useAuth();

  if (loading) return <div>Loading...</div>;
  if (!user) return <div>Please sign in</div>;

  return (
    <div>
      <p>Welcome, {user.displayName}!</p>
      <button onClick={signOut}>Sign Out</button>
    </div>
  );
}
```

## Firebase Configuration

### Setup

1. **Create Firebase Project**
   - Go to [Firebase Console](https://console.firebase.google.com/)
   - Create a new project
   - Enable Authentication

2. **Configure Authentication Providers**
   - Go to Authentication > Sign-in method
   - Enable Google provider
   - Configure OAuth consent screen

3. **Get Configuration**
   - Go to Project Settings > General
   - Add a web app
   - Copy the configuration object

4. **Environment Variables**
   Add to your `.env.local`:
   ```env
   NEXT_PUBLIC_FIREBASE_API_KEY=your_api_key
   NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
   NEXT_PUBLIC_FIREBASE_PROJECT_ID=your_project_id
   NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET=your_project.appspot.com
   NEXT_PUBLIC_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
   NEXT_PUBLIC_FIREBASE_APP_ID=your_app_id
   ```

### Firebase Utils

Utility functions for Firebase operations are available at `src/lib/firebase/firebaseUtils.ts`:

```typescript
import { 
  signInWithGoogle, 
  signOut, 
  getCurrentUser,
  getIdToken 
} from '@/lib/firebase/firebaseUtils';

// Sign in with Google
const user = await signInWithGoogle();

// Get current user
const currentUser = getCurrentUser();

// Get ID token for API calls
const token = await getIdToken();
```

## Authentication Flow

### Sign In Process

1. User clicks "Sign in with Google"
2. Firebase redirects to Google OAuth
3. User authorizes the application
4. Firebase returns user data and ID token
5. User state is updated in context
6. Token is stored for API calls

### Sign Out Process

1. User clicks "Sign out"
2. Firebase signs out the user
3. Local state is cleared
4. User is redirected to sign-in page

### Token Management

- ID tokens are automatically refreshed
- Tokens are included in API requests
- Expired tokens trigger re-authentication

## API Integration

### Protected Routes

API routes can be protected by verifying the Firebase ID token:

```typescript
// pages/api/protected-route.ts
import { verifyIdToken } from '@/lib/firebase/firebaseUtils';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  try {
    const token = req.headers.authorization?.replace('Bearer ', '');
    const decodedToken = await verifyIdToken(token);
    
    // Token is valid, proceed with request
    res.json({ message: 'Success', userId: decodedToken.uid });
  } catch (error) {
    res.status(401).json({ error: 'Unauthorized' });
  }
}
```

### Client-Side API Calls

Include the ID token in API requests:

```typescript
import { useAuth } from '@/lib/hooks/useAuth';

function MyComponent() {
  const { getIdToken } = useAuth();

  const callAPI = async () => {
    const token = await getIdToken();
    
    const response = await fetch('/api/protected-route', {
      headers: {
        'Authorization': `Bearer ${token}`
      }
    });
    
    const data = await response.json();
  };
}
```

## Security Considerations

### Token Security

- ID tokens are short-lived (1 hour)
- Tokens are automatically refreshed
- Never store tokens in localStorage
- Use secure HTTP-only cookies for sensitive data

### API Security

- Always verify tokens on the server
- Implement rate limiting
- Validate user permissions
- Log authentication events

### Data Protection

- Encrypt sensitive user data
- Implement proper access controls
- Regular security audits
- Follow GDPR/privacy regulations

## Error Handling

### Common Errors

1. **Invalid Token**
   - Token expired or malformed
   - Solution: Refresh token or re-authenticate

2. **Network Errors**
   - Firebase service unavailable
   - Solution: Retry with exponential backoff

3. **User Not Found**
   - User account deleted
   - Solution: Clear local state and redirect to sign-in

### Error Recovery

```typescript
import { useAuth } from '@/lib/hooks/useAuth';

function MyComponent() {
  const { user, error, retry } = useAuth();

  if (error) {
    return (
      <div>
        <p>Authentication error: {error.message}</p>
        <button onClick={retry}>Retry</button>
      </div>
    );
  }

  // Component content
}
```

## Testing

### Unit Tests

Test authentication hooks and utilities:

```typescript
import { renderHook } from '@testing-library/react';
import { useAuth } from '@/lib/hooks/useAuth';

test('returns user when authenticated', () => {
  const { result } = renderHook(() => useAuth());
  expect(result.current.user).toBeDefined();
});
```

### Integration Tests

Test authentication flow:

```typescript
import { render, screen, fireEvent } from '@testing-library/react';
import { AuthProvider } from '@/lib/contexts/AuthContext';
import SignInWithGoogle from '@/components/SignInWithGoogle';

test('signs in successfully', async () => {
  render(
    <AuthProvider>
      <SignInWithGoogle />
    </AuthProvider>
  );

  fireEvent.click(screen.getByText('Sign in with Google'));
  
  // Verify authentication state
});
```

## Troubleshooting

### Common Issues

1. **"Firebase not initialized"**
   - Check environment variables
   - Verify Firebase configuration

2. **"Google Sign-In not working"**
   - Check OAuth configuration
   - Verify domain settings

3. **"Token verification failed"**
   - Check server-side token verification
   - Ensure proper error handling

### Debug Mode

Enable debug logging:

```typescript
// In your Firebase config
import { getAuth, connectAuthEmulator } from 'firebase/auth';

if (process.env.NODE_ENV === 'development') {
  connectAuthEmulator(auth, 'http://localhost:9099');
}
```

## Migration Guide

### Upgrading Firebase

1. Update Firebase SDK version
2. Check breaking changes in changelog
3. Update authentication code if needed
4. Test thoroughly

### Adding New Providers

1. Enable provider in Firebase Console
2. Update authentication context
3. Add provider-specific components
4. Update documentation

## Support

For authentication issues:

1. Check Firebase Console for errors
2. Review browser console logs
3. Verify environment variables
4. Test with Firebase Auth emulator

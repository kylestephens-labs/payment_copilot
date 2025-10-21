# Components Documentation

This document provides detailed information about the React components available in the Payment Copilot application.

## Overview

All components are located in the `src/components/` directory and are built with TypeScript and Tailwind CSS.

## Available Components

### ImageUpload

A component for uploading and managing images.

**Location:** `src/components/ImageUpload.tsx`

**Props:**
- `onUpload: (file: File) => void` - Callback when file is uploaded
- `accept?: string` - Accepted file types (default: "image/*")
- `maxSize?: number` - Maximum file size in bytes
- `disabled?: boolean` - Whether the component is disabled

**Usage:**
```tsx
import ImageUpload from '@/components/ImageUpload';

function MyComponent() {
  const handleUpload = (file: File) => {
    console.log('Uploaded file:', file);
  };

  return (
    <ImageUpload
      onUpload={handleUpload}
      accept="image/*"
      maxSize={5 * 1024 * 1024} // 5MB
    />
  );
}
```

### SignInWithGoogle

A component for Google authentication using Firebase.

**Location:** `src/components/SignInWithGoogle.tsx`

**Props:**
- `onSuccess?: (user: User) => void` - Callback on successful sign-in
- `onError?: (error: Error) => void` - Callback on sign-in error
- `disabled?: boolean` - Whether the component is disabled
- `className?: string` - Additional CSS classes

**Usage:**
```tsx
import SignInWithGoogle from '@/components/SignInWithGoogle';

function LoginPage() {
  const handleSuccess = (user: User) => {
    console.log('User signed in:', user);
  };

  const handleError = (error: Error) => {
    console.error('Sign-in error:', error);
  };

  return (
    <SignInWithGoogle
      onSuccess={handleSuccess}
      onError={handleError}
      className="w-full"
    />
  );
}
```

### VoiceRecorder

A component for recording audio using the browser's MediaRecorder API.

**Location:** `src/components/VoiceRecorder.tsx`

**Props:**
- `onRecordingComplete: (audioBlob: Blob) => void` - Callback when recording is complete
- `onError?: (error: Error) => void` - Callback on recording error
- `maxDuration?: number` - Maximum recording duration in seconds
- `disabled?: boolean` - Whether the component is disabled

**Usage:**
```tsx
import VoiceRecorder from '@/components/VoiceRecorder';

function VoiceNoteComponent() {
  const handleRecordingComplete = (audioBlob: Blob) => {
    // Process the recorded audio
    console.log('Recording complete:', audioBlob);
  };

  return (
    <VoiceRecorder
      onRecordingComplete={handleRecordingComplete}
      maxDuration={60} // 1 minute
    />
  );
}
```

## Component Patterns

### Error Handling

All components include proper error handling with optional error callbacks:

```tsx
const handleError = (error: Error) => {
  console.error('Component error:', error);
  // Show user-friendly error message
};
```

### Loading States

Components that perform async operations include loading states:

```tsx
const [isLoading, setIsLoading] = useState(false);

// In component
{isLoading ? (
  <div className="animate-spin">Loading...</div>
) : (
  <button onClick={handleAction}>Action</button>
)}
```

### Accessibility

All components follow accessibility best practices:

- Proper ARIA labels
- Keyboard navigation support
- Screen reader compatibility
- Focus management

## Styling

Components use Tailwind CSS for styling with consistent design patterns:

### Color Scheme
- Primary: Blue variants (`blue-500`, `blue-600`, etc.)
- Success: Green variants (`green-500`, `green-600`, etc.)
- Error: Red variants (`red-500`, `red-600`, etc.)
- Warning: Yellow variants (`yellow-500`, `yellow-600`, etc.)

### Spacing
- Consistent padding and margins using Tailwind's spacing scale
- Responsive design with mobile-first approach

### Typography
- Consistent font sizes and weights
- Proper line heights for readability

## Customization

### Theme Customization

Components can be customized by modifying the Tailwind configuration:

```typescript
// tailwind.config.ts
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: {
          500: '#your-color',
          600: '#your-darker-color',
        },
      },
    },
  },
};
```

### Component Variants

Some components support different variants:

```tsx
// Example variant usage
<Button variant="primary" size="large">
  Click me
</Button>
```

## Testing

### Unit Tests

Components should be tested with React Testing Library:

```tsx
import { render, screen, fireEvent } from '@testing-library/react';
import ImageUpload from '@/components/ImageUpload';

test('renders upload button', () => {
  render(<ImageUpload onUpload={jest.fn()} />);
  expect(screen.getByText('Upload Image')).toBeInTheDocument();
});
```

### Integration Tests

Test components with their contexts and hooks:

```tsx
import { AuthProvider } from '@/lib/contexts/AuthContext';
import SignInWithGoogle from '@/components/SignInWithGoogle';

test('signs in with Google', async () => {
  render(
    <AuthProvider>
      <SignInWithGoogle onSuccess={jest.fn()} />
    </AuthProvider>
  );
  
  // Test sign-in flow
});
```

## Best Practices

1. **Props Interface**: Always define TypeScript interfaces for component props
2. **Default Props**: Use default parameters for optional props
3. **Error Boundaries**: Wrap components in error boundaries when appropriate
4. **Memoization**: Use `React.memo` for components that re-render frequently
5. **Custom Hooks**: Extract complex logic into custom hooks
6. **Documentation**: Document all props and usage examples

## Contributing

When adding new components:

1. Follow the established patterns
2. Include TypeScript types
3. Add proper error handling
4. Write unit tests
5. Update this documentation
6. Follow accessibility guidelines

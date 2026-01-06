# Firebase Authentication - Complete React Implementation

## Objective
Implement a complete authentication system using Firebase Authentication in a React application, including login, signup, password recovery, email verification, and global authentication state management.

## Technology Stack
- React 18+
- Firebase Authentication
- React Router v6
- TypeScript
- Context API for global state
- CSS Modules or Tailwind CSS for styling

## Requirements

### 1. Firebase Configuration
- Create a Firebase configuration file with environment variables
- Initialize Firebase app and authentication service
- Export reusable auth instance

### 2. Authentication Context Provider
Create a global authentication context that provides:
- Current user state
- Loading state
- Authentication methods (login, signup, logout, etc.)
- User profile data
- Error handling

### 3. Authentication Methods
Implement the following authentication flows:
- **Email/Password Signup** with email verification
- **Email/Password Login**
- **Forgot Password** flow with email reset link
- **Logout** functionality
- **Update Profile** (display name, photo URL)
- **Update Email**
- **Update Password**
- **Delete Account**

### 4. UI Components
Create the following screens/components:
- **Login Screen** - Email/password form with link to signup and forgot password
- **Signup Screen** - Registration form with validation
- **Forgot Password Screen** - Email input for password reset
- **Email Verification Notice** - Component shown when email is not verified
- **Profile/Account Screen** - Display user info and account management options
- **Protected Route Wrapper** - HOC or component to protect authenticated routes

### 5. Features & Best Practices
- Form validation (email format, password strength)
- Loading states during authentication operations
- Error handling with user-friendly messages
- Automatic redirect after login/logout
- Persistent authentication state (Firebase handles this)
- Email verification enforcement for sensitive operations
- Password reset confirmation
- Responsive design for all screen sizes

### 6. Security Considerations
- Never expose Firebase API keys in client code (use environment variables)
- Implement proper form validation
- Add CSRF protection where needed
- Implement rate limiting on sensitive operations
- Use Firebase Security Rules for Firestore/Storage if applicable

### 7. File Structure
```
src/
├── config/
│   └── firebase.ts
├── contexts/
│   └── AuthContext.tsx
├── components/
│   ├── auth/
│   │   ├── LoginForm.tsx
│   │   ├── SignupForm.tsx
│   │   ├── ForgotPasswordForm.tsx
│   │   └── EmailVerificationNotice.tsx
│   └── ProtectedRoute.tsx
├── pages/
│   ├── Login.tsx
│   ├── Signup.tsx
│   ├── ForgotPassword.tsx
│   └── Profile.tsx
├── hooks/
│   └── useAuth.ts
└── types/
    └── auth.types.ts
```

### 8. Expected Functionality

**AuthContext should expose:**
```typescript
interface AuthContextType {
  currentUser: User | null;
  loading: boolean;
  signup: (email: string, password: string, displayName: string) => Promise<void>;
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
  resetPassword: (email: string) => Promise<void>;
  updateUserProfile: (displayName: string, photoURL?: string) => Promise<void>;
  updateUserEmail: (email: string) => Promise<void>;
  updateUserPassword: (password: string) => Promise<void>;
  sendVerificationEmail: () => Promise<void>;
}
```

### 9. Environment Variables
```
VITE_FIREBASE_API_KEY=your_api_key
VITE_FIREBASE_AUTH_DOMAIN=your_auth_domain
VITE_FIREBASE_PROJECT_ID=your_project_id
VITE_FIREBASE_STORAGE_BUCKET=your_storage_bucket
VITE_FIREBASE_MESSAGING_SENDER_ID=your_sender_id
VITE_FIREBASE_APP_ID=your_app_id
```

## Deliverables
- Complete authentication system with all specified features
- Fully typed TypeScript implementation
- Responsive UI components
- Error handling and loading states
- Protected routes implementation
- README with setup instructions

## Testing Checklist
- [ ] User can sign up with email and password
- [ ] Email verification is sent after signup
- [ ] User can log in with correct credentials
- [ ] Login fails with incorrect credentials
- [ ] User can request password reset
- [ ] User receives password reset email
- [ ] User can update profile information
- [ ] User can log out successfully
- [ ] Protected routes redirect to login when not authenticated
- [ ] Authentication state persists across page refreshes
- [ ] All forms show proper validation errors
- [ ] Loading states display during async operations

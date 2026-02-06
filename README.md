# 🚀 Kinde Authentication Guide

This guide explains how to integrate **Kinde Authentication** in a **React (Vite + TypeScript)** application using **React Context API** and a custom hook — **no backend or APIs required**.

It also covers:
- Email & password login
- Google & Apple social login
- Password reset
- UI & branding customization  
**directly from the Kinde Dashboard (no code changes needed)**

---

## 🏷️ What is Kinde?

**Kinde** is a modern authentication and user management platform that fully manages:
- Login & Signup
- Social authentication
- Password resets
- User sessions
- OAuth tokens

Your React app only consumes auth state — **Kinde handles the rest**.

🔗 Official website: https://kinde.com  
🔗 Documentation: https://docs.kinde.com

---

## 📦 1. Install Dependency

```bash
npm install @kinde-oss/kinde-auth-react
```

---

## ⚙️ 2. Environment Variables

Create a `.env` file in your project root:

```env
VITE_KINDE_DOMAIN=your_kinde_domain
VITE_KINDE_CLIENT_ID=your_kinde_client_id
VITE_KINDE_REDIRECT_URI=http://localhost:5173
VITE_KINDE_LOGOUT_REDIRECT_URI=http://localhost:5173
```

📌 Get these values from  
**Kinde Dashboard → Settings → Applications**

🔗 App setup docs: https://docs.kinde.com/developer-tools/sdks/react-sdk/

---

## 📁 3. Folder Structure

```txt
src/
│── context/
│   └── AuthContext.tsx
│
│── hooks/
│   └── useAuth.ts
│
│── main.tsx
│── App.tsx
```

---

## 🧠 4. Auth Context (React Only)

This context:
- Wraps the app with `KindeProvider`
- Stores authentication state
- Exposes login, register, logout methods
- Shows a loader while auth initializes

🔗 Context usage reference:  
https://docs.kinde.com/developer-tools/sdks/react-sdk/#using-the-sdk

### `AuthContext.tsx`

```tsx
import React, { createContext, useEffect, useState } from 'react';
import { KindeProvider, useKindeAuth } from '@kinde-oss/kinde-auth-react';

interface AuthContextType {
  isAuthenticated: boolean;
  isLoading: boolean;
  user: Record<string, unknown> | null | undefined;
  login: () => void;
  register: () => void;
  logout: () => void;
  getToken: () => Promise<string | null>;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);
export { AuthContext };

interface AuthProviderProps {
  children: React.ReactNode;
}

export const AuthProvider: React.FC<AuthProviderProps> = ({ children }) => {
  const [loading, setLoading] = useState(true);

  const kindeConfig = {
    domain: import.meta.env.VITE_KINDE_DOMAIN,
    clientId: import.meta.env.VITE_KINDE_CLIENT_ID,
    redirectUri: import.meta.env.VITE_KINDE_REDIRECT_URI,
    logoutUri: import.meta.env.VITE_KINDE_LOGOUT_REDIRECT_URI,
  };

  useEffect(() => {
    const timer = setTimeout(() => setLoading(false), 1000);
    return () => clearTimeout(timer);
  }, []);

  if (loading) {
    return <div>Loading authentication...</div>;
  }

  return (
    <KindeProvider {...kindeConfig}>
      <AuthContextWrapper>{children}</AuthContextWrapper>
    </KindeProvider>
  );
};

const AuthContextWrapper: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const auth = useKindeAuth();

  return (
    <AuthContext.Provider
      value={{
        isAuthenticated: auth.isAuthenticated || false,
        isLoading: auth.isLoading || false,
        user: auth.user || null,
        login: auth.login || (() => {}),
        register: auth.register || (() => {}),
        logout: auth.logout || (() => {}),
        getToken: () =>
          auth.getToken
            ? auth.getToken().then(token => token || null)
            : Promise.resolve(null),
      }}
    >
      {children}
    </AuthContext.Provider>
  );
};
```

---

## 🎣 5. Custom Hook

### `useAuth.ts`

```ts
import { useContext } from 'react';
import { AuthContext } from '../context/AuthContext';

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

🔗 Hook reference: https://docs.kinde.com/developer-tools/sdks/react-sdk/#hooks

---

## 🧩 6. Wrap App with Provider

### `main.tsx`

```tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import { AuthProvider } from './context/AuthContext';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <AuthProvider>
      <App />
    </AuthProvider>
  </React.StrictMode>
);
```

---

## 💻 7. Using Auth in Components

```tsx
import { useAuth } from './hooks/useAuth';

const App = () => {
  const { isAuthenticated, user, login, register, logout } = useAuth();

  return (
    <div>
      {isAuthenticated ? (
        <>
          <p>Welcome {user?.email}</p>
          <button onClick={logout}>Logout</button>
        </>
      ) : (
        <>
          <button onClick={login}>Login</button>
          <button onClick={register}>Register</button>
        </>
      )}
    </div>
  );
};

export default App;
```

---

## 🔐 8. Login Methods (Email, Google, Apple)

👉 **No code required**

Enable from Kinde Dashboard:

- **Settings → Authentication → Sign-in methods**
- Enable:
  - Email & Password
  - Google
  - Apple

🔗 Social login docs:  
https://docs.kinde.com/authentication/social-logins/

---

## 🔁 9. Reset Password (No Code)

Kinde provides a built-in password reset flow.

- Login screen includes **Forgot password**
- User receives email reset link
- Password reset handled securely by Kinde

🔗 Password reset docs:  
https://docs.kinde.com/authentication/passwords/

---

## 🎨 10. UI & Branding Customization

You can fully customize the hosted auth UI.

Customizable items:
- Logo
- App name
- Colors
- Fonts
- Button styles
- Light / Dark mode

Steps:
- **Dashboard → Branding**
- Update styles
- Save & preview instantly

🔗 Branding docs:  
https://docs.kinde.com/branding/

---

## ⚡ Authentication Flow

1. App loads
2. Kinde initializes session
3. Hosted login UI opens
4. User logs in (Email / Google / Apple)
5. Redirects back to React app
6. Auth state available via Context

---

## 📚 Useful Documentation Links

- React SDK: https://docs.kinde.com/developer-tools/sdks/react-sdk/
- Authentication overview: https://docs.kinde.com/authentication/
- User profiles: https://docs.kinde.com/users/
- Tokens & sessions: https://docs.kinde.com/authentication/tokens/
- Security & compliance: https://docs.kinde.com/security/

---

## ✅ Done

Your React app now has **fully managed authentication** using **Kinde**,  
with **zero backend code** and **dashboard-driven configuration**.

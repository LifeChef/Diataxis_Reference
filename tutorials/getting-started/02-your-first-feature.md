# Tutorial: Your First Feature

> **Document Type**: Tutorial (Learning-Oriented)  
> **Audience**: Beginner  
> **Estimated Time**: 30 minutes

## Overview

Build a simple user profile display feature.

**What you'll learn:**
- Creating components
- Adding routes
- Writing tests

**What you'll build:**
A user profile page.

---

## Prerequisites

- [ ] Completed environment setup
- [ ] Development server running

---

## Step 1: Create Component

Create `src/components/UserProfile.jsx`:

```jsx
function UserProfile({ user }) {
  return (
    <div className="user-profile">
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
export default UserProfile;
```

---

## Step 2: Add Route

Update `src/App.jsx`:

```jsx
import UserProfile from './components/UserProfile';

// Add route
<Route path="/profile" element={<UserProfile />} />
```

---

## Step 3: Test Your Work

Visit http://localhost:3000/profile

---

## Summary

You've created your first feature!

**Next**: Explore advanced tutorials or how-to guides.

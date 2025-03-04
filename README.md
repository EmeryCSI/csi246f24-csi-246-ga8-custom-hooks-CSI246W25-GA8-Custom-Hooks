# Renton Technical College CSI-246

<div align="center">
  <img src="logo.jpg" alt="Logo">
  <h3 align="center">Guided Activity 8: CustomHooks</h3>
</div>

# Guided Activity 8: CustomHooks

## Overview

In this activity you will build custom hooks one at a time and test each by creating a corresponding page. You will create **three client‑side pages** that demonstrate different custom hook functionalities:

- **Data Fetching (`useFetch`)**: Fetch data from an API using an async function with try/catch.
- **Input Debouncing (`useDebounce`)**: Delay updating a value until the user stops typing (debouncing).
- **Local Storage Management (`useLocalStorage`)**: Synchronize state with localStorage so data persists across reloads.

Finally, you will create a **fourth page** that uses a server action which mimics the `useFetch` hook—returning an object with `data`, `loading`, and `error`.

---

## Part 1: Building and Testing the `useFetch` Hook

### Step 1.1: Create the `useFetch` Hook

Create a new file at `hooks/useFetch.ts`. In this hook, instead of using promise chaining (i.e. `.then().catch()`), we create an async function inside a useEffect to perform our fetch using try/catch.

```typescript
// hooks/useFetch.ts
import { useState, useEffect } from 'react';

/*
  The useFetch hook simplifies data fetching from an API endpoint.
  It accepts a URL as an argument and returns an object with:
    - data: The fetched data (or null if not yet available)
    - loading: A boolean flag indicating whether the fetch is in progress
    - error: A string containing an error message (if any error occurs)
  
  Instead of chaining .then()/.catch(), we use an async function with try/catch
  inside the useEffect for cleaner error handling.
*/
export function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    // Define an async function to fetch data
    const fetchData = async () => {
      try {
        // Start the fetch request
        const response = await fetch(url);
        // Check if the response is ok
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        // Parse the JSON data
        const jsonData = await response.json();
        // Update state with the fetched data
        setData(jsonData);
        setError(null);
      } catch (err: any) {
        // Update the error state if something went wrong
        setError(err.message);
      } finally {
        // Stop the loading state
        setLoading(false);
      }
    };

    // Call the async function defined above
    fetchData();
  }, [url]);

  return { data, loading, error };
}
```

### Step 1.2: Create the `useFetch` Test Page

Now create a page to test the hook. Create the file at `app/use-fetch/page.tsx`:

```typescript
// app/use-fetch/page.tsx
"use client";

import React from 'react';
import { useFetch } from '@/hooks/useFetch';

/*
  This page demonstrates the useFetch hook in action.
  It fetches data from a public API and displays:
    - A loading message while the data is being fetched.
    - An error message if the fetch fails.
    - The fetched data when available.
*/
export default function UseFetchPage() {
  // Change the URL below if needed; this sample uses a placeholder API.
  const { data, loading, error } = useFetch<any>('https://jsonplaceholder.typicode.com/todos/1');

  return (
    <main className="container mx-auto p-4">
      <h1 className="text-3xl font-bold mb-4">Data Fetch Example</h1>
      {loading && <p>Loading data...</p>}
      {error && <p className="text-red-500">Error: {error}</p>}
      {data && (
        <div className="bg-gray-100 p-4 rounded-md">
          <pre>{JSON.stringify(data, null, 2)}</pre>
        </div>
      )}
    </main>
  );
}
```

### Step 1.3: Testing

Run your development server:

```bash
npm run dev
```

Visit [http://localhost:3000/use-fetch](http://localhost:3000/use-fetch) in your browser and verify that the page displays a loading message, then the fetched data (or an error if there’s an issue).

---

## Part 2: Building and Testing the `useDebounce` Hook

### Step 2.1: Create the `useDebounce` Hook

Create a file at `hooks/useDebounce.ts` with the following code. Note that we include an explanation of debouncing.

> **What is Debouncing?**  
> Debouncing is a technique used to delay the execution of a function until a certain amount of time has passed since it was last invoked.  
> **Why use it?**  
> - **Performance:** It prevents unnecessary calls (for example, API requests) on every keystroke.  
> - **User Experience:** It ensures that actions (such as searches) only happen when the user has finished typing.

```typescript
// hooks/useDebounce.ts
import { useState, useEffect } from 'react';

/*
  The useDebounce hook takes in a value and a delay (in milliseconds) and returns a debounced value.
  The debounced value only updates after the specified delay has passed without further changes.
  
  This is useful for optimizing operations like API calls triggered by user input.
*/
export function useDebounce<T>(value: T, delay: number = 500): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    // Set a timer to update the debounced value after the delay
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    // Clear the timer if the value changes before the delay expires
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}
```

### Step 2.2: Create the `useDebounce` Test Page

Create a new page at `app/use-debounce/page.tsx` to test the hook:

```typescript
// app/use-debounce/page.tsx
"use client";

import React, { useState } from 'react';
import { useDebounce } from '@/hooks/useDebounce';

/*
  This page demonstrates the useDebounce hook.
  It includes an input field where:
    - The immediate value updates as the user types.
    - The debounced value updates only after the user stops typing for 500ms.
*/
export default function UseDebouncePage() {
  const [value, setValue] = useState('');
  const debouncedValue = useDebounce(value, 500);

  return (
    <main className="container mx-auto p-4">
      <h1 className="text-3xl font-bold mb-4">Debounce Example</h1>
      <label className="block mb-2">
        Enter text (updates will be debounced):
      </label>
      <input
        type="text"
        value={value}
        onChange={(e) => setValue(e.target.value)}
        className="w-full border p-2 rounded mb-4"
      />
      <p className="text-gray-700">
        <strong>Immediate Value:</strong> {value}
      </p>
      <p className="text-gray-700">
        <strong>Debounced Value (500ms delay):</strong> {debouncedValue}
      </p>
    </main>
  );
}
```

### Step 2.3: Testing

Visit [http://localhost:3000/use-debounce](http://localhost:3000/use-debounce) in your browser and test that as you type, the “Immediate Value” updates instantly while the “Debounced Value” only updates after a 500ms pause.

---

## Part 3: Building and Testing the `useLocalStorage` Hook

### Step 3.1: Create the `useLocalStorage` Hook

Create the file `hooks/useLocalStorage.ts` with the code below. This hook synchronizes state with localStorage, allowing data to persist across page reloads.

```typescript
// hooks/useLocalStorage.ts
import { useState, useEffect } from 'react';

/*
  The useLocalStorage hook manages state that is synchronized with the browser's localStorage.
  It accepts a key and an initial value. On initialization, it attempts to load the stored value.
  If none exists, it uses the initial value provided.
  Whenever the state updates, the hook saves the new value to localStorage.
*/
export function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') return initialValue;
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error('Error reading localStorage key “' + key + '”: ', error);
      return initialValue;
    }
  });

  useEffect(() => {
    if (typeof window !== 'undefined') {
      try {
        window.localStorage.setItem(key, JSON.stringify(storedValue));
      } catch (error) {
        console.error('Error setting localStorage key “' + key + '”: ', error);
      }
    }
  }, [key, storedValue]);

  return [storedValue, setStoredValue] as const;
}
```

### Step 3.2: Create the `useLocalStorage` Test Page

Create a new page at `app/use-localstorage/page.tsx`:

```typescript
// app/use-localstorage/page.tsx
"use client";

import React from 'react';
import { useLocalStorage } from '@/hooks/useLocalStorage';

/*
  This page demonstrates the useLocalStorage hook.
  It includes an input field tied to localStorage so that the entered data persists across page reloads.
*/
export default function UseLocalStoragePage() {
  const [name, setName] = useLocalStorage<string>('userName', '');

  return (
    <main className="container mx-auto p-4">
      <h1 className="text-3xl font-bold mb-4">Local Storage Example</h1>
      <label className="block mb-2">
        Enter your name (will persist across reloads):
      </label>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        className="w-full border p-2 rounded mb-4"
      />
      <p className="text-gray-700">
        Your stored name is: <strong>{name}</strong>
      </p>
    </main>
  );
}
```

### Step 3.3: Testing

Visit [http://localhost:3000/use-localstorage](http://localhost:3000/use-localstorage) and verify that the input value is saved in localStorage. Reload the page to ensure the stored value persists.

---

## Part 4: Building and Testing a Server Action that is similar to a custom hook

### Step 4.1: Create a Server Action Function

Below is the updated section with the changes applied:

---

Because hooks are client side you may find yourself wanting to build something similar for a Server Side Function. This is much simpler to execute on the server as server side components allow for async/await and run before the page is loaded.

Here is an example of what useFetch could look like for a server side component. Just as hooks state with use. Most server side functions start with get.

```typescript
// app/server-action/getData.ts
/*
  The getServerData function mimics the behavior of the useFetch hook,
  but it runs on the server. It fetches data from an API using async/await
  and returns an object with data, loading (always false on render), and error.
*/
export async function getServerData<T>(url: string): Promise<{ data: T | null; loading: boolean; error: string | null }> {
  try {
    const res = await fetch(url);
    if (!res.ok) {
      throw new Error('Network response was not ok');
    }
    const data = await res.json();
    return { data, loading: false, error: null };
  } catch (err: any) {
    return { data: null, loading: false, error: err.message };
  }
}
```

### Step 4.2: Create the Server Action Test Page

Now build a page that uses the above server action to fetch data. Create `app/server-action/page.tsx`:

```typescript
// app/server-action/page.tsx
import { getServerData } from './getData';

/*
  This is a server component that uses a server action to fetch data
  similarly to the useFetch hook. The getServerData function returns an object
  with data, loading, and error, which is then used to render the page.
*/
export default async function ServerActionPage() {
  // Use the server action function to fetch data from a public API.
  const { data, loading, error } = await getServerData<any>('https://jsonplaceholder.typicode.com/todos/1');

  return (
    <main className="container mx-auto p-4">
      <h1 className="text-3xl font-bold mb-4">Server Action Data Fetch</h1>
      {/* Although loading is always false at render time in a server component,
          we include it here to mirror the useFetch hook’s API */}
      {loading && <p>Loading data...</p>}
      {error && <p className="text-red-500">Error: {error}</p>}
      {data && (
        <div className="bg-gray-100 p-4 rounded-md">
          <pre>{JSON.stringify(data, null, 2)}</pre>
        </div>
      )}
    </main>
  );
}
```

---

## Final Steps: Running and Committing Your Changes

1. **Run Your Application:**  
   Use the following command to start your development server:
   ```bash
   npm run dev
   ```

2. **Test Each Page:**  
   - Visit [http://localhost:3000/use-fetch](http://localhost:3000/use-fetch) to test the useFetch hook.
   - Visit [http://localhost:3000/use-debounce](http://localhost:3000/use-debounce) to test the useDebounce hook.
   - Visit [http://localhost:3000/use-localstorage](http://localhost:3000/use-localstorage) to test the useLocalStorage hook.
   - Visit [http://localhost:3000/server-action](http://localhost:3000/server-action) to test the server action.

3. **Commit Your Changes:**

   ```bash
   git add .
   git commit -m "Guided Activity 8: Complete"
   git push
   ```

---

## Final Notes

- **Building Incrementally:**  
  This assignment is broken into steps where you build a hook and then create its test page. This incremental approach helps you test and understand each piece of functionality as you go.

- **Custom Hooks and Server Actions:**  
  By encapsulating reusable logic into hooks and comparing it to server actions, you gain insight into when to handle logic on the client versus the server—each with their own benefits and trade-offs.

- **Documentation:**  
  Notice the inline comments in each file. They explain not only how the code works, but also why certain patterns (like async/await and debouncing) are used.

Happy coding!

### Calculator Application

## Project Overview

A modern and responsive calculator application built using **SvelteKit and TypeScript**.  
The application provides basic arithmetic operations with keyboard support, calculation history, and a clean user interface.

---

## Features

- Addition, subtraction, multiplication, and division
- Real-time calculation display
- Keyboard input support
- Calculation history panel
- Responsive mobile layout
- Reusable Svelte components
- Error handling for invalid expressions
- Percentage calculations

---

## Technology Stack

- Frontend: SvelteKit
- Language: TypeScript
- Styling: CSS
- Icons: Lucide Svelte

---

## Project Structure

```text
calculator-app/
│
├── src/
│   │
│   ├── lib/
│   │   │
│   │   └── components/
│   │       └── Button.svelte
│   │
│   ├── routes/
│   │   ├── +page.svelte
│   │   └── +layout.svelte
│   │
│   ├── app.css
│   └── ...
│
└── screenshots/
    ├── calculator.png
    └── mobile.png

```

---

## Screenshots

### Calculator

<img width="1900" height="1001" alt="calculator" src="https://github.com/user-attachments/assets/a836d8f2-1736-4b44-be98-b209599ca3f1" />

---

### Mobile Responsive View

<img width="1296" height="927" alt="mobile" src="https://github.com/user-attachments/assets/bcd95e53-6753-4cd6-80ea-d1468cd907a7" />



---

## Usage

### Mouse Controls

- Click number buttons to enter values
- Select operators for calculations
- Use delete button to remove characters

### Keyboard Controls

Supported keys:

```
Numbers: 0-9
Operators: + - * /
Enter: Calculate
Backspace: Delete
Escape: Clear
Arrow Keys: Navigate buttons
```


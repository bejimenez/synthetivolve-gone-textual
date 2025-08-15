# Synthetivolve: TUI Edition - Project Plan

> **Your Personal Health & Wellness Engine, Reimagined for the Terminal**

This document outlines a re-architecture of Synthetivolve into a pure Python, terminal-based application using the Textual framework. The goal is to create a fast, keyboard-driven, and distraction-free interface for personal health tracking, deployed on a private VPS and accessed through a web browser or SSH.

## 🎯 Core Philosophy

The core philosophy remains unchanged: **Track adherence vs. effectiveness**. The application's strength lies in its intelligent, data-driven logic. This new architecture focuses on delivering that intelligence through the most efficient interface possible: a TUI.

## 🏛️ Core Architecture: The Unified Monolith

The fundamental shift is from a decoupled "frontend/backend" model to a **unified monolithic application**. The presentation layer (the TUI) and the business logic layer run in the same Python process. This dramatically simplifies the entire system.

*   **No API Layer:** The UI does not make HTTP requests to a backend. Instead, Textual widgets call Python functions directly from the core services layer.
*   **Stateful by Nature:** The application is a long-running, stateful process, allowing for instant UI updates and a highly responsive feel.
*   **Single-User Focus:** Security and authentication are radically simplified. Access control is managed at the server level (SSH access to your VPS), not within the application.

### Proposed Project Structure

```
synthetivolve-tui/
├── main.py                 # Main application entry point (runs the Textual app)
|
├── core/
│   ├── calculator.py       # TDEE, macro, and adjustment logic
│   └── analytics.py        # Rolling averages, trend analysis
|
├── database/
│   ├── models.py           # SQLAlchemy ORM models (User, WeightEntry, etc.)
│   └── engine.py           # Database session and engine setup
|
├── screens/
│   ├── dashboard.py        # Main dashboard screen
│   ├── log_entry.py        # Screen for logging weight, food, etc.
│   ├── history.py          # Screen for viewing historical data
│   └── settings.py         # Screen for managing goals and settings
|
└── widgets/
    ├── chart.py            # Custom widget for displaying simple text-based charts
    ├── summary_panel.py    # Reusable panel for showing daily summaries
    └── footer.py           # A custom footer showing keybindings
```

## 🛠️ Technical Stack

This stack is streamlined for a pure Python environment, removing all web and JavaScript dependencies.

### Core Stack
*   **Application Framework:** [Textual](https.textualize.io) - The primary framework for building the TUI.
*   **Database:** SQLite - Lightweight, file-based, and perfect for a single-user application.
*   **ORM:** SQLAlchemy - The Python SQL toolkit for all database interactions (standard synchronous mode is sufficient, but async can be used).
*   **Database Migrations:** Alembic - For version-controlling the database schema.
*   **Data Visualization:** [Sparklines](https://pypi.org/project/sparklines/) (or custom Textual widgets) - For creating simple, effective terminal-based charts and graphs.

### Removed Systems & Libraries
The following are **no longer needed** in this architecture:
*   **Web Frameworks:** FastAPI, Uvicorn
*   **API Validation:** Pydantic (though still useful for data modeling within the core logic)
*   **Authentication:** FastAPI-Users, python-jose, passlib
*   **JavaScript:** React, Vite, Node.js
*   **CSS:** Tailwind CSS
*   **JS Libraries:** Recharts, shadcn/ui, QuaggaJS

## ✨ TUI-Specific Design & UX Recommendations

A great TUI is not a website in a terminal; it's a unique experience built around efficiency.

1.  **Keyboard-First is Everything:**
    *   **Global Hotkeys:** Implement app-wide keybindings. For example, `d` always returns to the Dashboard, `l` opens the logging screen, and `q` quits.
    *   **Command Palette:** Implement a `Ctrl+P`-style command palette. This allows the user to type to find and execute any action (e.g., "log weight," "set new goal"), which is far faster than navigating menus.
    *   **Efficient Forms:** Ensure a logical tab order for all `Input` widgets. Use auto-selection and sensible defaults to minimize keystrokes.

2.  **Information Density and Layout:**
    *   **The "Golden Layout":** Use Textual's layout system to create a persistent layout: a `Header` for titles and status, a `Footer` for dynamic keybinding hints, and a main content area that screens can populate.
    *   **Focus Over Flash:** Use color and text styling purposefully. Your color-coding standards (green, yellow, red) are perfect here. Avoid clutter. Every character on the screen should serve a purpose.

3.  **Data Visualization in the Terminal:**
    *   **Sparklines for Trends:** Use the `sparklines` library to render a compact, expressive 7-day or 30-day weight trend directly on the dashboard.
    *   **Bar Charts for Macros:** Represent daily calorie and macro intake with simple, color-coded bar charts built with Textual widgets. You can use block characters (`█`) to create surprisingly effective visualizations.
    *   **Tables for History:** Use Textual's `DataTable` widget to display historical data. Make it sortable by column for easy analysis.

4.  **Adapting Features for the TUI:**
    *   **Barcode Scanning -> Quick-Add:** Since there is no camera, replace barcode scanning with a highly-optimized search-and-add function for your food database. As the user types, results should appear instantly.
    *   **"The Why Button":** This is still a great idea. It can be implemented as a hotkey (`?`) that opens a simple popup `ModalScreen` with the explanation text.

## 🔐 Security & Authentication (The Simple Way)

For a single-user application hosted on a private VPS, the security model is drastically simplified.

*   **Primary Security:** Your server's SSH access is the main security layer. If no one can log in to your VPS, no one can run the app.
*   **Application Lock (Optional):** For an extra layer of privacy (e.g., when accessing via `textual-web`), you can implement a simple startup screen that asks for a single, hard-coded password stored in an environment variable before loading the main TUI. There is no need for user tables, password hashing, or session management.

## 🚀 Deployment Plan

Deployment is fast and straightforward using `textual-web`.

1.  **Prerequisites on VPS:**
    *   Install Python 3.11+.
    *   Install your application's dependencies via `pip`.
    *   (Optional but Recommended) Set up a `systemd` service to ensure the application runs persistently and restarts on failure.

2.  **Running the App:**
    *   The `textual-web` command serves your Textual application over a web server, rendering it in a browser terminal.
    *   You can run it directly:
        ```bash
        textual web main.py
        ```
    *   To make it accessible, you can run it on `0.0.0.0` and put it behind a reverse proxy like Nginx or Caddy to handle SSL (HTTPS).

3.  **`systemd` Service Example (`/etc/systemd/system/synthetivolve.service`):**
    ```ini
    [Unit]
    Description=Synthetivolve TUI Application
    After=network.target

    [Service]
    User=your_username
    Group=your_username
    WorkingDirectory=/path/to/synthetivolve-tui
    ExecStart=/path/to/venv/bin/textual web main.py --host 0.0.0.0 --port 8080
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```
    This ensures the application is always running in the background, ready to be accessed.

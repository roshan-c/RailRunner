# Ticket to Ride Web App: MVP & Architecture (No Code Snippets)

## 1. Minimum Viable Product (MVP) Criteria

The goal of this MVP is to deliver a core, playable asynchronous Ticket to Ride experience with the unique natural language input feature powered by Gemini AI.

### **Core Functionality:**

1.  **User Authentication & Profiles:**
    *   **User Registration:** Allow new users to create accounts.
    *   **User Login/Logout:** Securely authenticate users.
    *   **Basic Profile:** Display username.
2.  **Game Lobby & Creation:**
    *   **Create Game:** Users can initiate a new game instance.
    *   **Join Game:** Users can join an existing game by invitation (e.g., sharing a game code/link) or from a list of open games.
    *   **Game List:** Display currently active or open games that the user can join.
    *   **Player Readiness:** Players indicate they are ready before the game starts.
3.  **Simplified Core Gameplay Loop:**
    *   **2-3 Player Support:** Focus on a smaller player count initially.
    *   **Basic Board Representation:** Visually display a simplified Ticket to Ride board with key cities and routes.
    *   **Natural Language Action Input:**
        *   A prominent text input field where players type their desired actions (e.g., "Draw two train cards," "Claim the route from London to Paris with 2 black cards").
        *   Feedback to the user if the command is unclear or invalid.
    *   **Train Card Drawing:**
        *   Players can use natural language to request drawing train cards from the deck.
        *   Display of face-up train cards and the option to draw from them via natural language (e.g., "Take the face-up red card").
    *   **Claiming Routes:**
        *   Players can use natural language to specify a route to claim (e.g., "Claim New York to Montreal with 3 orange cards").
        *   Validation: Check if the player has enough correct cards, if the route is available, etc.
    *   **Basic Scoring:** Automatically calculate and display points for claimed routes.
    *   **Turn Management:** Clearly indicate the current player and enforce turn order.
    *   **Asynchronous Play:** Game state persists, allowing players to take turns at their leisure, even if not online simultaneously.
    *   **End-of-Game Condition:** A simple end condition (e.g., when a player has X trains left, or all routes of a certain length are claimed).
    *   **Game State Persistence:** All game progress (board, player hands, scores) is saved and loaded correctly upon reconnecting.
4.  **Real-time Updates:**
    *   When a player takes an action, all other active players in the game room see the updates (board changes, hand sizes, scores, turn indicator) in near real-time.
5.  **Basic In-Game Chat:**
    *   A simple text-based chat window within each game room.

### **Technical & Deployment:**

1.  **Error Handling & Validation:**
    *   Robust validation of all player actions on the backend.
    *   Informative error messages displayed to the user for invalid moves or AI misinterpretations.
2.  **Deployment:**
    *   The entire application (frontend and backend) is deployed to a live environment and accessible via a URL.

### **Out of Scope for MVP (Future Iterations):**

*   Destination Tickets (drawing, completing, scoring, keeping)
*   Longest Continuous Path bonus
*   Globetrotter bonus
*   Advanced AI opponents
*   Spectator mode
*   Friend systems, direct invitations, or match-making
*   Complex animations or rich visual effects beyond basic board state
*   Extensive board variations or expansions
*   Undo/redo actions
*   Reconnecting to a game after a long period (beyond basic persistence)

---

## 2. Full Architecture

### 2.1. Overall Structure & Technology Choices

This architecture leverages the strengths of a reactive frontend framework (React), a real-time backend-as-a-service (Convex), and a powerful AI model (Gemini) for natural language processing.

*   **Frontend:** React with TypeScript, Vite (for development), and Tailwind CSS.
*   **Backend & Database:** Convex (TypeScript backend functions, reactive database).
*   **AI:** Google Gemini API (accessed from Convex backend).

**High-Level Flow:**

1.  **User interacts with Frontend:** A player types their action into a text input.
2.  **Frontend sends action to Convex:** The player's text command and relevant game/player IDs are sent to a Convex backend function (mutation).
3.  **Convex calls Gemini AI:** The Convex backend function constructs a detailed prompt (player command + current game context) and sends it to the Gemini API.
4.  **Gemini AI processes command:** Gemini interprets the natural language and returns a structured, machine-readable action (e.g., a JSON object).
5.  **Convex applies game logic:** The Convex backend function validates the structured action from Gemini against game rules and the current game state. If valid, it updates the game state in the Convex database.
6.  **Convex updates Frontend:** Due to Convex's real-time capabilities, all connected players' frontends automatically receive the updated game state and re-render.

### 2.2. File + Folder Structure

This structure aims for clear separation of concerns, modularity, and scalability.

```
ticket-to-ride-app/
├── .env                  # Environment variables (API keys, etc.)
├── package.json          # Project dependencies
├── tsconfig.json         # TypeScript configuration
├── vite.config.ts        # Vite (frontend build tool) configuration
├── convex.config.ts      # Convex client configuration
├── README.md             # Project documentation
│
├── convex/               # Backend and database definitions (Convex functions)
│   ├── auth.ts           # Convex authentication setup (user management)
│   ├── schema.ts         # Database schema definition (tables, fields, indexes)
│   ├── lib/              # Shared backend utility functions
│   │   ├── utils.ts      # General utility functions for backend logic
│   │   └── gemini.ts     # Wrapper for interacting with the Gemini API
│   ├── game/             # Core game logic and database interactions
│   │   ├── mutations.ts  # Functions that modify game state (e.g., create, join, take action)
│   │   ├── queries.ts    # Functions that read game state (e.g., get game, list games)
│   │   ├── constants.ts  # Game-specific static data (board layout, card types)
│   │   ├── logic.ts      # Pure functions implementing core game rules
│   │   └── actions.ts    # Interprets Gemini output and applies actions to game logic
│   ├── chat/             # Chat-related backend functions
│   │   ├── mutations.ts  # Function to send messages
│   │   └── queries.ts    # Function to retrieve messages
│   └── users/            # User profile management backend functions
│       ├── mutations.ts  # Functions to create/update user profiles
│       └── queries.ts    # Functions to retrieve user profiles
│
└── src/                  # Frontend (React application)
    ├── App.tsx           # Main application component
    ├── main.tsx          # Entry point for the React app
    ├── index.css         # Global styles or Tailwind CSS directives
    ├── lib/              # Frontend utility functions and API client
    │   ├── api.ts        # Auto-generated Convex API client for frontend use
    │   └── utils.ts      # General utility functions for frontend logic
    ├── assets/           # Static assets (images, icons for the board, cards)
    ├── components/       # Reusable React components
    │   ├── ui/           # Generic, presentation-focused UI elements (e.g., Button, Input)
    │   │   ├── Button.tsx
    │   │   ├── Input.tsx
    │   │   └── Dialog.tsx
    │   └── game/         # Game-specific UI components
    │       ├── GameBoard.tsx    # Renders the Ticket to Ride board
    │       ├── PlayerHand.tsx   # Displays player's train cards and relevant info
    │       ├── ActionInput.tsx  # The text input for natural language commands
    │       ├── GameLobby.tsx    # UI for creating and joining games
    │       └── ChatBox.tsx      # Renders in-game chat
    ├── hooks/            # Custom React hooks for logic reuse
    │   ├── useGame.ts    # Hook for fetching game data and initiating game actions
    │   └── useAuth.ts    # Hook for managing user authentication state
    ├── pages/            # Top-level view components (routes)
    │   ├── HomePage.tsx
    │   ├── GamePage.tsx
    │   └── LoginPage.tsx
    ├── context/          # React Context API providers (for global state if needed)
    │   └── AuthContext.tsx # Context for authentication status
    └── types/            # TypeScript type definitions for robust development
        ├── gameTypes.ts    # Types for game entities (Game, Player, Route, Card)
        ├── apiTypes.ts     # Types for Convex query/mutation inputs and outputs
        └── geminiTypes.ts  # Types for Gemini API request/response structures
```

### 2.3. What Each Part Does & Where State Lives

#### 2.3.1. Frontend (`src/`)

*   **Main Application Files:** Handle the initial rendering of your React application and global styling.
*   **Convex API Client (`lib/api.ts`):** This file is automatically generated by Convex and provides type-safe functions for your React components to call backend queries (read data) and mutations (write data). This is the primary way the frontend interacts with your backend.
*   **Utility Functions (`lib/utils.ts`):** General helper functions for frontend tasks like data formatting or UI logic that aren't specific to components.
*   **Assets (`assets/`):** Stores visual elements like images for the game board, train cards, player pieces, etc.
*   **Components (`components/`):**
    *   **UI Components (`ui/`):** Generic, reusable UI building blocks (e.g., buttons, text inputs, modal dialogs) that are independent of game logic.
    *   **Game Components (`game/`):** Specific UI elements that represent parts of the Ticket to Ride game. They use data fetched from Convex and trigger actions via Convex mutations.
        *   `GameBoard`: Visually renders the current state of the game board (cities, routes, claimed routes).
        *   `PlayerHand`: Displays the cards and trains a player currently holds.
        *   `ActionInput`: The crucial element where a player types their action in natural language. When submitted, it calls a Convex mutation.
*   **Hooks (`hooks/`):** Custom React hooks abstract and encapsulate logic.
    *   `useGame`: A central hook for a game page. It uses Convex queries to fetch the current game state reactively and provides functions (via Convex mutations) to perform game actions.
    *   `useAuth`: Manages the user's authentication status and provides methods for login/logout using Convex's authentication system.
*   **Pages (`pages/`):** Top-level components that represent distinct views or "pages" in your web application (e.g., the landing page, the game lobby, an individual game session).
*   **Context (`context/`):** If any global application state isn't managed by Convex's reactive queries (e.g., UI theme, user preferences not stored in the database), React Context provides a way to share it across components.
*   **Types (`types/`):** TypeScript interfaces and types for all data structures, ensuring strong typing and helping prevent common programming errors across frontend and backend.

    *   **State Location (Frontend):** The primary source of truth for game and user state is the **Convex database**, which is accessed via reactive Convex queries. Local UI state (e.g., the text currently typed into an input field, whether a modal is open) resides within individual React components using `useState`. Global, non-game-specific application state might reside in React Context.

#### 2.3.2. Backend (`convex/`)

This is where the core game logic, database interactions, and AI integration happen. All Convex files are written in TypeScript.

*   **Authentication (`auth.ts`):** Defines how users authenticate and specifies rules for accessing different backend functions. Convex handles user session management.
*   **Schema (`schema.ts`):** Defines the structure of your data in the Convex database. This ensures data consistency and allows Convex to optimize data access. It describes tables (e.g., `games`, `users`, `messages`) and their fields.
    *   **Example Schema (Conceptual):**
        *   `games` table: Stores the entire state of a game instance (board state, player data including hands/scores/trains, current player turn, game status, chat messages).
        *   `users` table: Stores user profiles (username, ID).
*   **Utilities (`lib/`):**
    *   `gemini.ts`: Contains the logic to make requests to the Google Gemini API. It handles sending the prompt and processing the raw API response.
    *   `utils.ts`: General helper functions for backend logic that are used across multiple Convex files.
*   **Game Logic (`game/`):**
    *   **Mutations (`mutations.ts`):** These are the server-side functions that modify data in the Convex database. They are the only way to write data. Key mutations include:
        *   `createGame`: Sets up a new game instance with initial board, decks, and players.
        *   `joinGame`: Adds a player to a game in the lobby.
        *   `processPlayerAction`: **This is the core of your AI integration.** It receives the natural language command from the frontend. It then constructs a detailed prompt for Gemini (including the player's text and relevant current game context). It sends this prompt to Gemini via `lib/gemini.ts`. Upon receiving Gemini's structured response, it uses `game/actions.ts` to validate and apply the intended game action, finally updating the `games` document in the database.
    *   **Queries (`queries.ts`):** These are server-side functions that read data from the Convex database. They are reactive, meaning any frontend component subscribed to them will automatically update when the underlying data changes in the database. Examples: getting the full state of a specific game, listing open games, or fetching a player's hand.
    *   **Constants (`constants.ts`):** Contains static, unchanging data specific to Ticket to Ride rules and board (e.g., predefined routes, city names, card types).
    *   **Logic (`logic.ts`):** Contains "pure" functions that encapsulate the core mathematical and logical rules of Ticket to Ride. These functions take an input game state and an action, and deterministically return a *new* game state, without directly interacting with the database. This separation simplifies testing and ensures game rule consistency.
    *   **Actions (`actions.ts`):** Acts as an intermediary. It takes the structured action parsed by Gemini, performs specific game rule validations (e.g., "does the player have enough cards for *this* specific route?"), and then calls the appropriate pure functions from `game/logic.ts` to calculate the resulting new game state.
*   **Chat Logic (`chat/`):** Functions for sending and retrieving in-game chat messages.
*   **User Logic (`users/`):** Functions for managing user profiles (creating, updating, fetching).

    *   **State Location (Backend):** All authoritative game state (board, player hands, scores, turn, etc.) and user data resides exclusively in the **Convex database**. Convex mutation functions are the sole entry points for modifying this state.

### 2.4. How Services Connect

1.  **Frontend to Convex:**
    *   The React frontend uses the Convex client SDK (library) to interact with the Convex backend.
    *   **Real-time data:** Frontend components "subscribe" to Convex Queries. Whenever the data returned by a query changes in the Convex database, the frontend automatically receives the update and re-renders the affected components. This powers the real-time game board and score updates.
    *   **Player Actions:** When a player makes a move (e.g., types a command), the frontend calls a Convex Mutation function on the backend, passing the necessary input (like the natural language text).

2.  **Convex to Gemini AI:**
    *   A specific Convex Mutation function (e.g., `processPlayerAction` within `game/mutations.ts`) acts as the gateway to the Gemini API.
    *   Within this Convex function, an HTTP request is made directly to the Google Gemini API endpoint. This request includes the carefully crafted prompt (natural language command plus game context) and API authentication.
    *   The response from Gemini (the structured interpretation of the player's command) is then received and processed by the Convex function.

3.  **Convex Internal Connections:**
    *   Convex functions can securely call other Convex functions within the same backend deployment. This allows for modularity, where a mutation might call a helper query or a game logic function to compose a complete action.
    *   The core game rules are encapsulated in pure functions in `game/logic.ts`. These functions are called by `game/actions.ts` (which is in turn called by a mutation) to compute the next state based on an action, without directly touching the database. This separation ensures game logic is consistent and testable.

BM Habits — Versions A and B
BM Habits is a Flutter habit-tracking app developed for the study “Development and Usability Evaluation of a Mobile Habit-Tracking Application with AI.” It supports two study groups: Version A (habit tracking without AI) and Version B (the same tracker with AI recommendations and a chat coach).

Version A and Version B
Both versions share the same habit-tracking experience. This lets a study compare the effect of AI support while keeping the core app consistent.

Area	Version A — tracking only	Version B — AI-assisted
Create and manage habits	Yes	Yes
Build habits and avoid habits	Yes	Yes
Daily success or lapse check-ins	Yes	Yes
Record reasons and notes for a lapse	Yes	Yes
Review missed or unrecorded days	Yes	Yes
Current and best streaks, completion rate, and history	Yes	Yes
Personalised AI risk estimates and tips	No	Yes, after at least 7 days of history
Conversational AI habit coach	No	Yes
Version A — tracking only
Version A gives participants the core tools to build or quit habits: create a routine, record daily outcomes, reflect on lapses, and review progress. It does not show AI recommendations or provide the AI coach. Habit information is still stored in the participant's account and synced through Firebase.

Version B — AI-assisted tracking
Version B includes every Version A feature and adds two AI tools:

Personalised tips: On the Today screen, the app can show short tips and a risk level for habits. It sends up to 60 days of success/missed history, habit type, category, target weekday, and self-reported lapse reasons from the previous 14 days to the recommendation service. At least 7 days of history are required before a recommendation is requested.
AI habit coach: Open the coach from the Today screen to ask for ideas or discuss a challenge. The app sends summaries for up to 12 habits and up to the 12 most recent chat messages. The conversation is kept in memory while the app is open.
AI tools need the separately deployed backend configured below. If the service is unreachable, recommendations may not appear and the coach may report that it cannot connect. The chat backend can also enforce a temporary usage quota.

Assigning a study version
The app reads the participant's group from the group field on the Firestore document users/{uid}. Set it to A or B for each participant. If the field is missing or the group cannot be read, the app defaults to Version A. A developer-only override is available for local UI checks:

flutter run --dart-define=FORCE_GROUP=B
Do not use this override for participant builds. It is intended only for testing the Version B interface. The study group controls whether AI recommendations and the coach entry point are shown. Version A participants see only the shared habit-tracking features.

Shared habit-tracking features
Create build habits (actions to do) and quit habits (actions to avoid).
Choose suggested daily habits during first-time setup or create custom ones.
Record daily outcomes. A lapse can include one or more reasons and an optional note.
Review unrecorded days from the last week; edit older dates in a habit's calendar where available. Unrecorded days remain unknown until reviewed.
View current and best streaks, a 30-day completion rate, recent activity, a seven-day strip, and a five-week heatmap.
Create an account and sign in with email and password. Each user's habits are stored under their own Firestore account path and synced live across devices.
The app does not configure scheduled or push reminders.
Technology and data
Flutter and Dart
Firebase Authentication for email/password sign-in
Cloud Firestore for user group assignment, habits, and check-in records
A separately deployed HTTP API for AI recommendations and coach chat
Each habit entry stores a success value, recording timestamp, optional reasons, and optional note. The date key identifies the habit day, including entries added during a retrospective review. Reasons are self-reported by the user.

Requirements
Flutter SDK compatible with Dart >=3.3.0 <4.0.0
Firebase project configured for the target platform
For Version B: a reachable AI service implementing POST /recommend and POST /chat
Setup and run
From the repository root:

flutter pub get
flutter run
The repository includes Android and iOS platform folders. If regenerating platform scaffolding is necessary, run flutter create . from the repository root and review the generated changes before committing them.

To run the automated test suite:

flutter test
Firebase platform configuration is provided in the platform-specific files and lib/firebase_options.dart. Use the Firebase project configuration intended for your app and deployment environment.

AI service configuration
The default AI API URL is configured in lib/config.dart. Override it when running or building the app with the AI_SERVICE_URL Dart define:

flutter run --dart-define=AI_SERVICE_URL=https://habit-tracking-application-with-ai.onrender.com
The base URL should point to the deployed service; the app appends /recommend and /chat for the corresponding requests. The request client is implemented in lib/services/ai_service.dart.

Project structure
lib/
  main.dart                    App startup, authentication gate, navigation
  config.dart                  AI service URL
  models/habit.dart            Habit and entry models, streak calculations
  services/
    auth_service.dart          Firebase Authentication
    habit_store.dart           Firestore persistence and study group loading
    ai_service.dart            AI recommendation and chat API client
  screens/
    today_screen.dart          Daily progress, check-ins, AI entry points
    history_screen.dart        Habit history and activity views
    ai_coach_screen.dart       Conversational AI coach
  widgets/
    ai_tips_section.dart       Personalised recommendation cards

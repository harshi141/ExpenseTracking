# ExpenseFlow — Smart Expense Tracker
## College Mini-Project & External Viva Documentation

---

### 1. Project Title & Overview
**Project Title:** ExpenseFlow — Smart Expense Tracker  
**Platform:** Android (API 26+)  
**Language:** Kotlin  
**User Interface:** Android XML Layouts with Material Design 3  
**Database & Auth:** Firebase Authentication (Email/Password & Google Sign-In) & Cloud Firestore
**Machine Learning:** Google ML Kit Text Recognition (On-Device Receipt OCR)  
**APIs Used:** Frankfurter Currency Conversion API  

ExpenseFlow is a clean, modern, and user-friendly Android financial management app designed to help users track daily expenses, scan receipt totals via OCR, convert foreign currency expenses to INR, analyze spending trends, and manage monthly budget limits with automatic threshold alerts.

---

### 2. Project Objective
The main objective of ExpenseFlow is to simplify personal financial tracking by:
1. Automatically extracting monetary values from printed receipts using Machine Learning.
2. Supporting foreign currency transactions with live conversion to INR.
3. Preventing overspending through real-time monthly budget alerts at 80% and 100% thresholds.
4. Synchronizing user expenses securely across devices via Firebase Cloud Storage.

---

### 3. Technologies & Architecture Used

- **Kotlin:** Primary language for clean, concise, beginner-friendly code.
- **Activity & XML Layout Architecture:** Simple Activity-based structure with ViewBinding, helper classes, models, and RecyclerView adapters.
- **Firebase Authentication:** Secure email/password login and user session persistence.
- **Cloud Firestore:** Document-based NoSQL database for real-time cloud data sync.
- **Google ML Kit Text Recognition:** On-device Optical Character Recognition (OCR) to extract prices from receipt photos.
- **Frankfurter Currency API:** Free REST API for real-time conversion rates to Indian Rupees (INR).
- **Material Design 3:** Modern UI styling, cards, progress bars, and MaterialAlertDialogBuilder dialogs.

---

### 4. Key Application Features

1. **User Authentication:** Email/Password registration, login, persistent session, and guest mode.
2. **Financial Dashboard:** Real-time summary of total spent, monthly budget, remaining budget, and percentage usage.
3. **Expense Management (CRUD):** Add, view, search, edit, and delete expenses with title, category, date, amount, currency, and notes.
4. **Receipt Scanning (ML Kit OCR):** Selects receipt image from gallery, extracts numeric values, and pre-fills the largest detected amount.
5. **Multi-Currency Support:** Auto-converts USD, EUR, GBP, AED, SGD, JPY, AUD, and CAD to INR using the live Frankfurter API.
6. **Monthly Budget Alerts:** 
   - **80% Used:** Amber warning dialog ("Warning! You have used 80% of your budget").
   - **100% Limit Reached:** Red alert dialog ("Budget Limit Reached!").
7. **Monthly Analytics:** Category-wise percentage distribution with horizontal progress bars.

---

### 5. Application Workflow

```text
[ Splash Screen ]
       │
       ▼
[ Authentication Check ] ────► [ Login / Register Screen ]
       │                                   │
       ├───────────────────────────────────┘
       ▼
[ Main Dashboard ]
   ├── Total Spent, Budget, Usage %
   ├── Recent Transactions List
   └── Navigation Bar / Quick Actions
         │
         ├──► [ Add / Edit Expense ] ──► (Optional) Scan Receipt via ML Kit OCR
         │                                 └──► Auto Currency Conversion to INR
         │
         ├──► [ All Expenses List ] ───► Search & Category Filter
         │
         ├──► [ Analytics Screen ] ────► Category Breakdown & Progress Bars
         │
         ├──► [ Budget Settings ] ────► Set Monthly Limit & Trigger Alerts
         │
         └──► [ Profile Screen ] ─────► User Details & Sign Out
```

---

### 6. Technical Components Explanation

#### A. Firebase Authentication & Firestore
- **Auth:** Uses `FirebaseAuth.getInstance().signInWithEmailAndPassword()` and `createUserWithEmailAndPassword()`.
- **Firestore Schema:**
  - `users/{userId}/expenses/{expenseId}`: Stores expense entries.
  - `users/{userId}/budgets/{monthKey}`: Stores monthly budget limits and alert flags.

#### B. Receipt Scanning via ML Kit OCR
- Uses `com.google.mlkit.vision.text.TextRecognition` to process images from photo picker.
- Extracts lines of text, applies Regex `(?:(?:₹|Rs\.?|INR|\$)\s*)?(\d+(?:\.\d{1,2})?)` to extract prices, selects the largest valid monetary value, and pre-fills the amount field.

#### C. Frankfurter Currency Conversion
- Sends GET HTTP request to `https://api.frankfurter.app/latest?from={CUR}&to=INR`.
- Calculates `amountINR = originalAmount * exchangeRate`.
- Runs off the UI main thread using Kotlin Coroutines (`Dispatchers.IO`).

#### D. Budget Usage & Progress Calculation
- **Formula:** $\text{Usage Percentage} = \left( \frac{\text{Total Monthly Spent}}{\text{Monthly Budget Limit}} \right) \times 100$
- **Color Logic:**
  - `< 80%`: Green (`#10B981`)
  - `80% - 99%`: Amber (`#F59E0B`)
  - `>= 100%`: Red (`#EF4444`)

---

### 7. Common College Viva Questions & Answers

**Q1: Why did you choose Firebase for Authentication and Database?**  
*Answer:* Firebase provides instant user authentication, real-time data synchronization, offline caching, and user isolation through unique User IDs (UIDs).

**Q2: How does the ML Kit Receipt Scanning feature work?**  
*Answer:* ML Kit performs on-device optical character recognition (OCR) on receipt images selected from the device gallery. It detects text blocks, runs regular expressions to find price patterns, identifies the largest valid monetary amount, and pre-fills it into the expense field.

**Q3: How is currency conversion handled?**  
*Answer:* When a foreign currency like USD or EUR is selected, the app makes an asynchronous HTTP GET request to the Frankfurter REST API (`api.frankfurter.app`) on a background thread. It fetches the current exchange rate to INR and calculates the equivalent Indian Rupee amount before saving.

**Q4: How are budget alerts prevented from repeating endlessly?**  
*Answer:* The app maintains boolean flags (`alert80Shown` and `alert100Shown`) inside Firestore and local preferences for each monthly budget record (`YYYY-MM`). Once an alert dialog is acknowledged by the user, the flag is set to `true` to avoid repeating the popup.

**Q5: How does the app handle offline operation or missing internet?**  
*Answer:* The app includes a dual-layer storage architecture (`FirebaseHelper`). If internet is unavailable or Firebase is offline, it saves and retrieves records locally using SharedPreferences JSON storage, ensuring zero app crashes.

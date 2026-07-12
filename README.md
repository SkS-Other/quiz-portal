# Quiz Portal

[Русский](./README.ru.md) | [English](./README.md)

A lightweight quiz platform starter built with Node.js + Express + MongoDB + EJS.

## 🚀 What's implemented

- **Project initialization**
    - `npm init`
    - Installed:  
      `express`, `mongoose`, `dotenv`, `ejs`, `express-ejs-layouts`,  
      `express-session`, `connect-mongo`, `passport`, `passport-local`,  
      `connect-flash`, `helmet`, `morgan`, `method-override`, `express-validator`, `bcrypt`.

- **Directory structure**

```

─ public/ # static assets (css, js, images)
- scripts/ # seed script
─ src/
│ ├─ config/ # db.js, passport.js
│ ├─ controllers/ # route logic (auth, users, quizzes, admin, admin, index)
│    └─ api/ # quizzes
│ ├─ middleware/ # ensureAuthenticated, requireAdmin, requireOwnerOrAdmin
│ ├─ models/ # User, Quiz, Question, Attempt
│ ├─ routes/ # index, auth, users, quizzes, admin, home
│    └─ api/ # quizzes
│ └─ views/ # EJS templates: layouts, partials, pages
│    ├─ layouts/ # main
│    ├─ pages/ # auth, users, quizzes, admin, home
│    └─ partials/ # header, footer, flash
─ .env # MONGO\_URI, SESSION\_SECRET, PORT
─ app.js # server entry point
─ package.json

````

- **Database (MongoDB + Mongoose)**
- Connection via `config/db.js`, URI in `.env`
- Models with schemas and `ref` relations:
    - `Attempt` (user, quiz, answers, score, timestamps)
    - `Question` (quiz, text, type, options [text, isCorrect], timestamps)
    - `User` (name, email – unique, passwordHash, role, timestamps)
    - `Quiz` (title, description, user, isPublic, accessToken, stats, timestamps)

- **Security and utilities**
- `helmet()` for HTTP headers
- `morgan('dev')` for logging
- `express.urlencoded()`, `express.json()`, `method-override('_method')`
- `express-session` + `connect-mongo` — sessions in MongoDB
- `passport.initialize()`, `passport.session()` — local strategy
- `connect-flash` — `error`/`success` flash messages
- `express-validator` — form validation

- **EJS template engine + Layouts & Partials**
- `express-ejs-layouts` with a shared `views/layouts/main.ejs`
- Partials:
    - `views/partials/header.ejs` — navigation, respects `currentUser`
    - `views/partials/flash.ejs`  — renders errors/successes from `req.flash`
    - `views/partials/footer.ejs`
- In `app.js`:
  ```js
  app.use((req, res, next) => {
    res.locals.currentUser = req.user || null;
    res.locals.successMsg  = req.flash('success');
    res.locals.errorMsg    = req.flash('error');
    next();
  });
  ```

- **Authentication and authorization**
- **Registration** (`GET /register`, `POST /register`)
    - Validation via `express-validator`
    - Hashing with `bcrypt.hash(password, 10)`
- **Login** (`GET /login`, `POST /login`) — Passport-Local
- **Logout** (`GET /logout`)
- **Passport-config** (`config/passport.js`)

- **User profile**
- **View** `/profile`, `/user/:id`
- **Edit** `/user/:id/edit`, `PUT /user/:id`

- **Main routes**
- `/`, `/register`, `/login`, `/logout`
- `/quizzes`, `/quizzes/new`, `/quizzes/:id`, `/quizzes/:id/edit`, …
- `/admin/*`
- REST API under `/api/*`

### Viewing and taking quizzes

- **Routes**  
  `GET /quizzes/:id` — render the quiz page  
  `POST /quizzes/:id` — count answers, save an `Attempt`, update `Quiz.stats`  
  `GET /quizzes/:id/result` — results page

- **Controllers** (`src/controllers/quizzesController.js`)  
  `showQuiz`, `submitQuizAnswers`, `showQuizResult`
- Support for types `single`, `multiple`, `text`, `truefalse`
- Updates `stats.attemptsCount` and `stats.averageScore`

- **Attempt model**  
  `answers: Map<Schema.Types.Mixed>` for arbitrary answers

### Editing and deleting quizzes

- **Controllers**:  
  `updateQuiz` (flash + redirect),  
  `deleteQuiz` (cascade: `Question`, `Attempt`)
- **Middleware**: `requireOwnerOrAdmin`

### Admin panel and admin features

- **Middleware**  
  `requireAdmin` blocks `/admin/*` when `req.user.role !== 'admin'`

- **Routes** (`src/routes/admin.js`)
- `GET /admin` — stats (number of users, quizzes
- **Users**
    - `GET  /admin/users` — list
    - `DELETE /admin/users/:id` — delete + cascade (quizzes → questions/attempts; user's attempts)
    - `POST   /admin/users/:id/toggle-admin` — toggle role `user ↔ admin`
- **Quizzes**
    - `GET    /admin/quizzes` — list
    - `DELETE /admin/quizzes/:id` — delete + cascade (questions, attempts)

- **Controllers** (`src/controllers/adminController.js`)
- `showDashboard`, `listUsers`, `deleteUser`, `toggleAdmin`
- `listQuizzes`, `deleteQuiz`
- Everything via EJS → flash → redirect

- **EJS templates** (`views/pages/admin/…`)
- `dashboard.ejs`
- `users.ejs`
- `quizzes.ejs`

- **Flash & method-override**
- Using `<form …?_method=DELETE>` + `method-override('_method')`
- Flash messages on every action

---

## 📦 Installation and running

1. Clone the repo
2. Install dependencies

  ```bash
  npm install
````

3. Create a `.env` next to `app.js`:

   ```dotenv
   MONGO_URI=mongodb://localhost:27017/quizdb
   SESSION_SECRET=any_string
   PORT=3000
   NODE_ENV=development
   ```
4. Start the server

   ```bash
   npm start
   # or
   node app.js
   ```
5. Open in the browser:
   [http://localhost:3000](http://localhost:3000)

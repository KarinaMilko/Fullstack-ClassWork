## 🌟 Fullstack-ClassWork: User & Task Management Platform

This repository demonstrates a **full-stack application** consisting of a **React/Redux** frontend and a **Node.js/Express** backend with **Sequelize** and a **PostgreSQL** database. It manages user entities (`User`) and their associated tasks (`Task`), implementing advanced features like data validation, file uploads, and specific controller logic (e.g., PUT/PATCH).

---

### ⚙️ Technologies Used

| Category | Technology | Description |
| :--- | :--- | :--- |
| **Backend** | Node.js, Express.js | REST API server foundation. |
| **Database** | PostgreSQL | Primary relational data store. |
| **ORM** | Sequelize, `sequelize-cli` | Models, migrations, associations, and data manipulation. |
| **Authentication** | `bcrypt` | Hashing user passwords securely. |
| **File Uploads** | `multer` | Handling `multipart/form-data` for image uploads. |
| **Frontend** | React, Redux Toolkit | Single Page Application (SPA) for the client. |
| **Forms/Validation** | Formik, Yup | Client-side form management and schema validation. |

---

### 🏛️ Backend Architecture (Server)

The server logic is structured around Sequelize Models, Migrations, Seeders, Controllers, and Middleware.

#### 1. Database Schema and Models

The application uses two main models with a **one-to-many relationship** (`User.hasMany(Task)` / `Task.belongsTo(User)`).

* **`User` Model (`server/models/user.js`)**:
    * **Fields**: `nickname` (unique, length validation), `email` (unique, `isEmail`), `tel` (unique, custom regex validation `^\+380\d{9}$|^0\d{9}$`), `passwHash` (auto-hashed using a `set` hook with `bcrypt`), `birthday` (date validation: `isBefore: new Date()`), `gender`, `role`, `image` (path to profile photo).
* **`Task` Model (`server/models/task.js`)**:
    * **Fields**: `body` (cannot be empty), `deadline` (date validation: `isAfter` yesterday).
    * **Association**: Belongs to a `User` via `user_id`.

#### 2. Migrations and Constraints

Advanced constraints are enforced directly in PostgreSQL via migrations (in addition to Sequelize model validation):

* **`User` Constraints**:
    * `tel`: Check constraint using regex for phone format.
    * `birthday`: Check constraint to ensure `birthday <= CURRENT_DATE`.
* **`Task` Constraints**:
    * `body`: Check constraint to ensure body is not an empty string.
    * `deadline`: Check constraint to ensure `deadline >= CURRENT_DATE`.

#### 3. Controllers and Logic

* **`usersController`**: Implements standard CRUD operations for users.
    * **`createUser`**: Handles both regular data and file uploads (`req.file`), calculates the image path, and uses `lodash.omit` to remove sensitive (`passwHash`) and timestamps before sending the response.
    * **`getUsers`**: Supports **pagination** via `limit` and `offset` using `page` and `results` query parameters.
    * **`updateOrCreateUserById` (PUT)**: Implements a unique two-step controller logic: it tries to update the user; if no user is found (`!updatedUser`), it modifies the `req.body` and calls `next()` to pass control to the `createUser` controller to perform the creation.
    * **`getUsersTasks`**: Finds a user and uses the Sequelize association method (`foundUser.getTasks()`) to retrieve all linked tasks.
* **`tasksController`**:
    * **`getTasks`**: Fetches all tasks and includes the associated `User`'s nickname using the `include` option.

#### 4. Routing and File Handling

* **Routes (`server/routes/usersRouter.js`)**:
    * `POST /`: Create User (includes `uploadUserPhoto` middleware).
    * `GET /`: Get all Users (with pagination support).
    * `PUT /:userId`: Update or Create User (using the chained controller logic).
    * `GET /:userId/tasks`: Get tasks for a specific user.
    * `PATCH /:userId/images`: Update user profile image (uses `uploadUserPhoto`).
* **Middleware (`server/middleware/upload.js`)**:
    * Uses `multer` with `diskStorage` to save files to the configured static folder (e.g., `public/images`).
    * Implements `fileFilter` to restrict uploads to `jpeg/png/gif` MIME types and returns a `415 Unsupported Media Type` error for invalid types.
* **Error Handling (`server/middleware/errorHandlers.js`)**: Centralized error middleware:
    * `multerErrorHandler`: Catches `multer.MulterError`.
    * `dbErrorHandler`: Catches **Sequelize `ValidationError`** and returns detailed errors with status `422 Unprocessable Entity`. Also handles general `BaseError` from Sequelize.
    * `errorHandler`: General fallback for other HTTP errors.

---

### 🖥️ Frontend Client (React/Redux)

The client provides the interface for interacting with the backend API.

* **State Management**: Uses **Redux Toolkit** to manage the `usersData` slice.
    * **Thunks**: `createUserThunk`, `getUsersThunk`, `removeUserThunk` handle asynchronous API calls via `axios`.
    * **API Client (`client/src/api/index.js`)**: Sends data, automatically setting `Content-Type: multipart/form-data` when submitting the `FormData` object.
* **User Form (`client/src/components/forms/UserForm/index.jsx`)**:
    * Uses **Formik** and **Yup** (`USER_VALIDATION_SCHEMA`).
    * **File Handling**: Converts form data (including the `userPhoto` file) into a **`FormData`** object for submission, ensuring the file is correctly sent to the `multer` middleware on the server.
* **User List (`client/src/components/UsersList/index.jsx`)**:
    * Fetches users on component mount.
    * Displays user data and profile images, constructing the image URL using the static folder path: `http://localhost:5000/${u.image}`.
    * Allows deletion of users.

---

### 🚀 Setup and Installation

Follow these steps to get the project running locally.

#### 1. Configuration and Dependencies

1.  Clone the repository:
    ```bash
    git clone [repository-link]
    cd Fullstack-ClassWork
    ```
2.  Install **Client** dependencies:
    ```bash
    cd client && npm install
    ```
3.  Install **Server** dependencies:
    ```bash
    cd server && npm install
    ```
4.  Create a **`.env`** file in the project root (`Fullstack-ClassWork/`) using a template:
    ```env
    # --- PostgreSQL Database Parameters ---
    DB_HOST=localhost
    DB_PORT=5432
    DB_USER=postgres
    DB_PASSWORD=... # <-- Enter your DB password
    DB_NAME=task_manager_development

    # --- Security and File Parameters ---
    SALT_ROUNDS=10
    STATIC_FOLDER=public # Path for saving uploaded files (images)
    PORT=5000
    ```

#### 2. Database Setup

1.  Navigate to the **Server** directory:
    ```bash
    cd server
    ```
2.  Run Sequelize migrations to create the tables:
    ```bash
    npx sequelize db:migrate
    ```
3.  Populate the tables with test data:
    ```bash
    npx sequelize db:seed:all
    ```

#### 3. Running the Application

1.  Start the **Backend Server** (staying in the `server` directory):
    ```bash
    cd server
    npm run dev
    ```
2.  Start the **Frontend Client** (in a new terminal, navigate to `client`):
    ```bash
    cd client
    npm run dev
    ```

The client application will typically be available at `http://localhost:5173/` (or similar port), and the API server will be listening at `http://localhost:5000/api`.

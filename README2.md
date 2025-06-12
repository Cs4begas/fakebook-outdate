# Codebase Explanation

This application is a single-page application (SPA) built using **React**. It leverages several other libraries and design patterns to create a functional user experience with role-based access control and persisted user sessions.

## 1. Core Technologies & Structure

*   **React:** The foundation for building the user interface with reusable components.
*   **React Router (`react-router-dom`):** Handles client-side routing, enabling navigation between different views without full page reloads.
*   **Redux (`redux`, `react-redux`):** Used for managing the application's global state. This is crucial for user authentication status, user information, and other shared data.
*   **Ant Design (`antd`):** A UI library providing a comprehensive set of pre-built React components (forms, buttons, layout grids, navigation bars, etc.) for a consistent and professional look and feel.
*   **Axios:** A promise-based HTTP client used for making API requests to a backend server.
*   **`jwt-decode`:** A utility library to decode JSON Web Tokens (JWTs), which are often used for transmitting user authentication information.
*   **Directory Structure:** The primary application code resides within the `src/` directory:
    *   `components/`: Contains reusable UI components (e.g., `NavBar`).
    *   `pages/`: Top-level components representing different views or "pages" of the application (e.g., `Home.js`, `Login.js`).
    *   `redux/`: Houses all Redux-related code, including actions, reducers, and store configuration.
    *   `config/`: Stores configuration files, such as API service settings (`api.service.js`) and role definitions (`roles.js`).
    *   `images/`: Contains static image assets used within the application.
    *   `App.js`: The main application component that sets up the overall layout and integrates the routing structure.
    *   `index.js`: The entry point of the React application, responsible for rendering the `App` component into the HTML DOM.

## 2. Application Startup and Initialization (`src/index.js`)

*   The application begins execution when `ReactDOM.render()` is called in `src/index.js`.
*   It renders the main `App` component, which serves as the root of the component tree.
*   The `App` component is wrapped with two important higher-order components:
    *   `<Provider store={store}>`: Provided by `react-redux`, this makes the Redux `store` (configured in `src/redux/store/store.js`) accessible to any component in the application that needs to interact with the global state (typically by using the `connect` higher-order component).
    *   `<BrowserRouter>`: Provided by `react-router-dom`, this enables client-side routing capabilities, allowing the URL to change and different components to render without requesting new pages from the server.
*   Global CSS styles from `index.css` and Ant Design's default styles (`antd/dist/antd.css`) are imported to ensure consistent styling across the application.

## 3. State Management with Redux

Redux serves as a centralized store for the application's global state.

*   **Store (`src/redux/store/store.js`):**
    *   This file is responsible for creating and configuring the Redux store.
    *   **State Persistence:** A key feature implemented here is the persistence of the Redux store to the browser's `localStorage`.
        *   `loadState()`: On application startup, this function attempts to retrieve any previously saved state from `localStorage` (stored under the key `'store'`). If found, this state is used to initialize the Redux store, allowing user sessions and other application states to persist across browser refreshes or new sessions.
        *   `saveState()`: The store is configured to automatically save its entire state to `localStorage` whenever any part of the state changes (`store.subscribe`).
    *   The root reducer (imported from `src/redux/reducers/reducers.js`) is passed to `createStore`, which combines the logic of all individual reducers.
    *   The setup also includes boilerplate for integrating with the Redux DevTools browser extension, which is invaluable for debugging state changes.

*   **Reducers (`src/redux/reducers/`):**
    *   `reducers.js`: This file uses `combineReducers` from Redux to merge different reducers into a single root reducer. In this application, it primarily combines the `userReducer`, mapping it to the `user` slice of the overall application state.
    *   `userReducer.js`: This reducer is specifically responsible for managing the state related to the current user.
        *   **Initial State Determination:** When the `userReducer` initializes, it checks `localStorage` for an item named `'ACCESS_TOKEN'`.
            *   If an access token exists, it's decoded using `jwt-decode`. The payload of the token (which typically contains user details like `id`, `role`, `name`, `profilePic`) is then used to set the initial state of the `user` slice.
            *   If no access token is found in `localStorage`, the user is assumed to be a "guest," and their `role` is set to `"guest"` by default.
        *   **Handling Actions:** The reducer defines how the `user` state should change in response to specific actions:
            *   `LOGIN_USER`: Dispatched (ideally) after a successful login, this action provides user details (`id`, `role`, `name`, `profilePic`) in its payload. The reducer updates the `user` state with this information.
            *   `LOGOUT_USER`: Dispatched when the user logs out or their session expires. The reducer resets the `user` state, primarily by setting the `role` back to `"guest"`.

*   **Actions (`src/redux/actions/actions.js` - *existence inferred by usage*):**
    *   Although not directly read, this file is expected to define:
        *   **Action Type Constants:** Strings like `LOGIN_USER` and `LOGOUT_USER` that serve as unique identifiers for actions.
        *   **Action Creators:** Functions (e.g., `login(userData)`, `logout()`) that components can call to create and dispatch actions to the Redux store.

## 4. Routing Mechanism

The application uses React Router to manage navigation and control access to different sections based on user roles.

*   **Main Router Setup (`src/App.js`):**
    *   The `App` component uses a `<Switch>` component from React Router within its `Content` area. The `<Switch>` renders the first child `<Route>` or `<Redirect>` that matches the current URL.
    *   The core routing logic, including role-based access control, is encapsulated within the `PrivateRoute` component.

*   **Role Configuration (`src/config/roles.js`):**
    *   This file is central to the application's authorization strategy.
    *   It defines a `components` object that maps symbolic names (e.g., `login`, `home`) to their corresponding React component names (e.g., `'Login'`, `'Home'`) and their designated URL paths (e.g., `'/login'`, `'/'`).
    *   It then defines different user roles (e.g., `admin`, `user`, `guest`). For each role, an array of `routes` is specified, indicating which parts of the application users with that role are allowed to access. These `routes` are typically constructed using the entries from the `components` object.

*   **Private Route (`src/components/routes/PrivateRoute.js`):**
    *   This component acts as a gatekeeper for routes that require authentication or specific roles.
    *   It receives the current user's `role` as a prop (passed down from `App.js`, which sources it from the Redux store).
    *   In its `componentDidMount` lifecycle method (and potentially `componentDidUpdate` if the role can change dynamically), it uses the `role` to look up the `allowedRoutes` from the `rolesConfig`.
    *   It then dynamically generates React Router `<Route>` components for each `allowedRoute`. The actual page component for each route is resolved using a mapping like `allRoutes[route.component]`. `allRoutes` is presumed to be an import from `src/components/routes/index.js`, which should export all page components.
    *   If the user's `role` is `"guest"`, they are automatically redirected to the `'/login'` path. If the `role` is not yet defined (e.g., during the initial application load before the Redux state is fully initialized), it also redirects to login as a protective measure.

*   **Route Definitions (`src/components/routes/index.js` - *existence inferred by usage in PrivateRoute.js*):**
    *   This file is expected to act as a central module that exports all the page-level components (e.g., `Home`, `Login`, `Profile`, `Signup`, `ChangePassword`, `Friend`). This allows `PrivateRoute` to dynamically render them based on the string component names specified in `roles.js`.

## 5. API Communication (`src/config/api.service.js`)

All backend communication is intended to be handled through a globally configured Axios instance.

*   **Global Axios Configuration:**
    *   `axios.defaults.baseURL = 'http://localhost:8080'`: Sets a base URL for all API requests, so individual calls only need to specify the endpoint path.
    *   `axios.defaults.withCredentials = true`: Configures Axios to send cookies with cross-origin requests, which is often necessary for session management if the backend relies on cookies.

*   **Request Interceptor:**
    *   This interceptor function is executed before each API request is sent.
    *   It differentiates between `UNPROTECTED_PATHS` (like login or registration endpoints) and protected paths.
    *   For protected paths, it retrieves an access token (presumably a JWT) from `localStorage` (where it's stored under the key `TOKEN`, as defined in `src/config/constants.js`).
    *   This token is then added to the `Authorization` header of the outgoing request, typically prefixed with `Bearer `.

*   **Response Interceptor:**
    *   This interceptor function processes API responses before they are passed back to the calling code.
    *   Its primary role is to handle authentication errors, specifically `401 Unauthorized` responses.
    *   If a `401` error occurs on a request to a protected resource:
        *   A message "Session expire, redirect to login" is logged to the console.
        *   The `TOKEN` is removed from `localStorage`, invalidating the client-side session.
        *   The `LOGOUT_USER` action is dispatched to the Redux store. This updates the application state to reflect that the user is no longer authenticated (e.g., sets `user.role` to `"guest"`).
        *   The error is then re-thrown, allowing the original promise chain in the calling code to also handle the error (e.g., by displaying a message to the user).

## 6. Login Process (`src/pages/authentication/Login.js`)

The login page provides the interface for users to authenticate.

*   **UI:** It uses Ant Design's `Form`, `Input`, and `Button` components to create the login form (username and password fields).
*   **Submission Logic (`handleSubmit`):**
    *   When the form is submitted, it validates the fields.
    *   **Current Implementation Detail:** The provided code shows that the login credentials (`login_username`, `login_password`) are sent using the browser's native `fetch` API directly to an external PHP script: `http://localhost/secure-code/cors2.php`.
    *   The response from this `fetch` call is displayed in a browser `alert()`.
*   **Potential Discrepancy & Missing Integration:**
    *   This direct `fetch` call deviates from the application's standard API communication pattern, which uses the configured Axios instance (`api.service.js`).
    *   More critically, the `handleSubmit` function in the `Login.js` snippet **does not appear to perform crucial steps for integrating with the application's authentication system:**
        1.  **Dispatch Redux Action:** It does not dispatch the `login` action (which should be available via `mapDispatchToProps`) to update the Redux store with the authenticated user's details (token, role, name, etc.).
        2.  **Store Access Token:** It does not show any logic for retrieving an access token from the `cors2.php` response and storing it in `localStorage` (under the `TOKEN` key).
    *   Without these steps, even if `cors2.php` successfully validates the credentials, the rest of the React application (Redux state, Axios interceptors for subsequent requests, `PrivateRoute` logic) will not recognize the user as logged in.

## 7. Key UI Components

*   **Navigation Bar (`src/components/navbar/NavBar.js`):**
    *   This component is connected to the Redux store to access the current user's information (`this.props.user`, which includes `name`, `profilePic`, and `role`).
    *   It displays the application logo.
    *   If a user is logged in, it shows their profile picture and name.
    *   It features a dropdown menu with navigation links:
        *   "ดูรายชื่อเพื่อน" (View Friends List) - links to `/friends`.
        *   "เปลี่ยนรหัสผ่าน" (Change Password) - links to `/changepassword`.
        *   "ออกจากระบบ" (Logout):
            *   When clicked, it calls `this.props.logout()` (which is expected to dispatch the Redux `logout` action).
            *   It then programmatically navigates the user to the home page (`'/'`) using `this.props.history.push('/')`.
            *   Finally, it forces a full page reload (`window.location.reload(true)`). This ensures a clean state, and if the user is now a "guest" due to the logout, `PrivateRoute` will likely redirect them to the login page.

*   **Home Page (`src/pages/Home.js`):**
    *   This component is also connected to the Redux store to access the current user's details.
    *   In `componentDidMount`, it initializes some local component state (`owner`) based on the user data from Redux.
    *   It then uses the configured `Axios` instance (`api.service.js`) to make a `GET` request to the `/feed` API endpoint to fetch data (presumably a list of posts).
    *   The fetched data is stored in the component's local state (`this.state.postList`).
    *   The `render` method is responsible for displaying these posts, though the snippet shows this part as a placeholder (`<Row></Row>`).

This explanation provides a comprehensive overview of the React application's architecture, state management, routing, API interaction, and key components based on the provided codebase snippets.

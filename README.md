<div align="center">

<img src="assets/mockups/hero.png" alt="KampanYES app: home, flyer and category screens" width="100%">

# KampanYES

**A digital-flyer platform: shoppers browse supermarket and retail folders for their city in a mobile app, while staff run brands, stores, flyers and push notifications from a web admin.**

![Flutter](https://img.shields.io/badge/Flutter-02569B?logo=flutter&logoColor=white)
![Dart](https://img.shields.io/badge/Dart-0175C2?logo=dart&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-339933?logo=nodedotjs&logoColor=white)
![Express](https://img.shields.io/badge/Express_5-000000?logo=express&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?logo=mongodb&logoColor=white)
![React](https://img.shields.io/badge/React_18-61DAFB?logo=react&logoColor=black)
![Vite](https://img.shields.io/badge/Vite-646CFF?logo=vite&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase_FCM-FFCA28?logo=firebase&logoColor=black)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?logo=githubactions&logoColor=white)

*Three codebases, one product: a Flutter mobile app, a Node.js REST API and a React admin dashboard.*

</div>

> [!NOTE]
> This is a **showcase repository**. The production source code is private. Everything here
> (screens, mock-ups, diagrams) was captured from the real apps running against a **mock API with
> fictional data**: made-up brands, cities, users and flyers. No real customer data is shown.

---

## The app in motion

<div align="center">
  <img src="assets/demo.gif" alt="Walkthrough of the KampanYES app: home feed, flyer, store info, categories, favourites, settings and dark mode" width="320">
  <br>
  <sub>Real Flutter build on the iOS simulator: home feed → flyer → store info → categories → favourites → settings → dark mode.</sub>
</div>

---

## What it does

Shoppers pick a city and see the current promotional flyers of nearby retailers, the digital version of the
brochures that land in the mailbox. They flip through a flyer, look at the store details, and save flyers and
stores they care about. Staff manage all of this content, order the flyers per city and send push notifications
from the admin dashboard.

<img src="assets/mockups/features.png" alt="Shopper features" width="100%">

| For shoppers (mobile app) | For staff (admin dashboard) |
|---|---|
| **City-based feed**: flyers for the selected city, paginated, with search by name, category or location | **Dashboard**: totals for stores, users, categories, collections and flyers, plus click analytics per city and per flyer |
| **Flyer viewer**: flip through the pages of a flyer, with validity dates | **Brands and store locations**: chains, and the folders/locations that belong to them |
| **Store info**: opening hours, address, description, website link and map | **Flyers by city**: upload flyers (PDFs can be converted to page images), reorder them per city |
| **Categories** to browse by type of shop | **Categories, countries and cities** management |
| **Favourites** for flyers and stores | **Users**: list, search, suspend and reactivate accounts |
| **Accounts**: email and password, Google and Apple sign-in, OTP-based password reset, or continue as guest | **Push notifications** to all app users |
| **Push notifications** and a forced-update check on launch | **Legal pages** (terms, privacy policy) with a rich-text editor |
| **English, German and Dutch**, light and dark theme | Admin-only login |

### Admin dashboard

<img src="assets/mockups/admin.png" alt="Admin dashboard screens" width="100%">

### Languages

<img src="assets/mockups/languages.png" alt="The app in English, German and Dutch" width="100%">

### Dark mode, settings and sign-in

<img src="assets/mockups/more.png" alt="Dark mode, settings, language picker and sign-in" width="100%">

---

## Architecture

The mobile app and the admin are thin clients of a single REST API. The API owns the data (MongoDB), the
authentication (JWT) and the integrations (Firebase push, SMTP email). Images are uploaded to the API and served
statically.

```mermaid
flowchart LR
    subgraph clients["Clients"]
        direction TB
        app["Mobile app<br/>Flutter, GetX<br/>iOS and Android"]
        admin["Admin dashboard<br/>React, Vite, TanStack Query"]
    end
    subgraph sys["KampanYES API"]
        direction TB
        api["REST API /api/v1<br/>Node.js, Express 5, TypeScript"]
        mw["Middleware<br/>helmet, rate limit, JWT auth,<br/>role permissions, validation"]
        ctrl["Controllers and services<br/>flyers, folders, stores, favourites,<br/>notifications, analytics"]
        db[("MongoDB<br/>via Mongoose")]
        files["Uploaded images<br/>served statically"]
    end
    subgraph ext["External systems"]
        fcm["Firebase Cloud Messaging<br/>push notifications"]
        smtp["SMTP<br/>OTP and reset emails"]
        gmaps["Google Maps<br/>store locations"]
        idp["Google and Apple<br/>sign-in"]
    end
    app --> api
    admin --> api
    api --> mw --> ctrl --> db
    ctrl --> files
    ctrl -.-> fcm
    ctrl -.-> smtp
    app -.-> gmaps
    admin -.-> gmaps
    app -.-> idp
    fcm -.-> app
```

### Domain model

The core hierarchy has three levels. Counts and favourite flags are computed at read time (Mongoose virtuals)
instead of being stored.

> UI vocabulary and API vocabulary differ: the admin calls a chain a **Brand** and a location a **Store**;
> in the API they are `Store` and `Folder`.

```mermaid
erDiagram
    COUNTRY ||--o{ CITY : has
    STORE ||--o{ FOLDER : "has (brand to location)"
    FOLDER ||--o{ FLYER : contains
    CATEGORY ||--o{ STORE : classifies
    CATEGORY ||--o{ FOLDER : classifies
    CITY }o--o{ FOLDER : "offered in"
    USER }o--o| CITY : "selected city"
    USER ||--o{ FAVORITE : saves
    FAVORITE }o--o| FLYER : "flyer"
    FAVORITE }o--o| STORE : "store"
    FAVORITE }o--o| FOLDER : "folder"
    USER ||--o{ OTP_MASTER : "password reset"
    NOTIFICATION }o--o{ USER : "pushed to"
    ANALYTICS }o--o| FLYER : "click events"
    ANALYTICS }o--o| CITY : "click events"
```

---

## Tech stack

| Layer | Technology |
|---|---|
| **Mobile app** | Flutter and Dart, GetX (a controller and a view per screen), `dio` for HTTP, `get_storage`, secure storage for tokens, `cached_network_image`, Google Maps, `intl` localisation (en, de, nl) |
| **Admin dashboard** | React 18, TypeScript, Vite 6, React Router 7, TanStack Query, Zustand, Ant Design 5, Tailwind 4, react-hook-form with Zod, dnd-kit for drag-to-reorder, ApexCharts, react-quill |
| **Backend** | Node.js, Express 5, TypeScript, Mongoose, Helmet, express-rate-limit, express-validator |
| **Auth** | JWT access and refresh tokens, role-based permissions (user, admin), optional auth so guests can browse, Google and Apple sign-in, OTP email flows, httpOnly cookies for the admin |
| **Data** | MongoDB (hosted cluster), GeoJSON locations, virtual populated fields |
| **Integrations** | Firebase Cloud Messaging, Firebase Auth, Nodemailer (SMTP), Google Maps |
| **Tooling** | pnpm, Biome, lefthook and commitlint (admin), Postman collections for the API |
| **CI/CD** | GitHub Actions (see below), PM2 on a VPS |

---

## Engineering highlights

- **Guests and members share one API.** An `optionalAuth` middleware downgrades to guest on any failure, so
  anonymous users can browse flyers while signed-in users also get their `isFavorite` flags populated.
- **Computed, not duplicated, data.** Folder flyer counts and favourite flags are Mongoose virtuals resolved at
  read time, so there is no counter to keep in sync.
- **Token handling on both clients.** The mobile app keeps bearer tokens in Keychain/Keystore-backed secure
  storage and refreshes them on a 401 before retrying the request. The admin uses httpOnly cookies, and its
  login endpoint rejects non-admin accounts server-side.
- **Batched analytics.** The app records flyer clicks locally and sends them to the API in batches (every
  minute, or once 20 events have piled up), keeping the feed fast and the traffic low.
- **Pagination and search.** The feed is paginated in pages of 20 with infinite scroll; search matches
  category names first and falls back to partial flyer names.
- **Geo-aware queries.** Locations are stored as GeoJSON points, and the API can filter the feed by distance.
- **Typed admin data layer.** One axios wrapper unwraps the API envelope, flattens validation errors and logs
  out on 401; a service module per domain feeds TanStack Query hooks.
- **Force-update gate and localisation.** The app checks the store version on launch, and is fully translated
  to English, German and Dutch with light and dark themes.
- **Security basics on the API.** Helmet security headers, a global rate limit (100 requests per minute) and
  request validation on the routes.

---

## CI/CD

```mermaid
flowchart LR
    dev["Developer<br/>push to main"] --> gha["GitHub Actions"]
    gha --> a["Admin: install, build<br/>CI also runs on develop,<br/>feature branches and PRs"]
    gha --> b["Backend: SSH to the server"]
    a --> sync["Sync build to the VPS"]
    b --> pm2["git pull, npm install,<br/>npm run build,<br/>pm2 restart"]
    app["Mobile app<br/>built and released<br/>through the App Store<br/>and Play Store"]
```

---

## Screens

### Mobile app

<table>
  <tr>
    <td><img src="assets/screens/app-home.png" width="190"></td>
    <td><img src="assets/screens/app-home-scrolled.png" width="190"></td>
    <td><img src="assets/screens/app-flyer.png" width="190"></td>
    <td><img src="assets/screens/app-store.png" width="190"></td>
  </tr>
  <tr>
    <td align="center"><sub>Home feed</sub></td>
    <td align="center"><sub>Scrolling the feed</sub></td>
    <td align="center"><sub>Flyer viewer</sub></td>
    <td align="center"><sub>Store info</sub></td>
  </tr>
  <tr>
    <td><img src="assets/screens/app-categories.png" width="190"></td>
    <td><img src="assets/screens/app-category-flyers.png" width="190"></td>
    <td><img src="assets/screens/app-favourites.png" width="190"></td>
    <td><img src="assets/screens/app-favourite-stores.png" width="190"></td>
  </tr>
  <tr>
    <td align="center"><sub>Categories</sub></td>
    <td align="center"><sub>Flyers of a category</sub></td>
    <td align="center"><sub>Favourite flyers</sub></td>
    <td align="center"><sub>Favourite stores</sub></td>
  </tr>
  <tr>
    <td><img src="assets/screens/app-settings.png" width="190"></td>
    <td><img src="assets/screens/app-location.png" width="190"></td>
    <td><img src="assets/screens/app-drawer.png" width="190"></td>
    <td><img src="assets/screens/app-terms.png" width="190"></td>
  </tr>
  <tr>
    <td align="center"><sub>Settings</sub></td>
    <td align="center"><sub>City picker</sub></td>
    <td align="center"><sub>Side menu</sub></td>
    <td align="center"><sub>Terms and conditions</sub></td>
  </tr>
</table>

### Admin dashboard

<table>
  <tr>
    <td><img src="assets/screens/admin-overview.png" width="380"></td>
    <td><img src="assets/screens/admin-brands.png" width="380"></td>
  </tr>
  <tr>
    <td align="center"><sub>Dashboard</sub></td>
    <td align="center"><sub>Brands</sub></td>
  </tr>
  <tr>
    <td><img src="assets/screens/admin-cities.png" width="380"></td>
    <td><img src="assets/screens/admin-flyers.png" width="380"></td>
  </tr>
  <tr>
    <td align="center"><sub>Flyers by city</sub></td>
    <td align="center"><sub>Flyers of a city</sub></td>
  </tr>
  <tr>
    <td><img src="assets/screens/admin-categories.png" width="380"></td>
    <td><img src="assets/screens/admin-users.png" width="380"></td>
  </tr>
  <tr>
    <td align="center"><sub>Categories</sub></td>
    <td align="center"><sub>Users</sub></td>
  </tr>
  <tr>
    <td><img src="assets/screens/admin-countries.png" width="380"></td>
    <td><img src="assets/screens/admin-notifications.png" width="380"></td>
  </tr>
  <tr>
    <td align="center"><sub>Countries and cities</sub></td>
    <td align="center"><sub>Push announcements</sub></td>
  </tr>
</table>

---

<div align="center">

Made by **[Mennova](https://mennova.nl/cases)** · more projects on **[mennova.nl/cases](https://mennova.nl/cases)**

</div>

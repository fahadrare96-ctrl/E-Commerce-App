Maison — E-Commerce Product Listing App
Status: ✅ Ready to run

Quick Start
No build step. No dependencies. Just open one file:
bashopen index.html
Or serve it locally (recommended for full localStorage support):
bashnpx serve .
# or
# then visit http://localhost:3000 

What's Inside
maison-shop/
│
├── index.html                  ← The entire app (self-contained, open this)
│
└── src/                        ← Reference modules (for React Native migration)
    ├── database/
    │   └── db.js               ← SQLite-mirroring data layer
    ├── hooks/
    │   └── useProducts.js      ← Custom React hooks (useProducts, useCart, useWishlist)
    └── components/
        └── ProductCard.jsx     ← Product card component (JSX reference)

Note: index.html is fully self-contained — it does not import from src/. The src/ folder contains the same logic as standalone modules, ready to drop into a React Native / Expo project.


Features
FeatureDetailsProduct gridResponsive CSS auto-fill grid, 2-col on mobileCategory filtersChips re-fetch from the DB layer on clickSearchClient-side filter with 200 ms debounceSortPrice low→high, high→low, top ratedSkeleton loading8-card shimmer animation while data loadsError + retryFull error state with "Try again" button (4% simulated error rate)Add to cartAsync DB write, badge counter, bump animation, spinner on buttonWishlistHeart toggle, persisted in localStorageProduct detailClick any card to open a modal with full descriptionCart drawerSlide-in drawer with live totals and item removalToast notificationsStacked, auto-dismiss after 2.7 sKeyboard shortcutsEsc closes modal/drawer · Cmd/Ctrl+K focuses searchLow stock warning"Only N left" badge when stock ≤ 8

Architecture
index.html — the runnable app
Everything needed to run in a browser is inlined here: CSS, HTML, and ~500 lines of vanilla JS. The JS is structured in three clear layers:
DB layer (dbQuery, getProducts, addToCart, …)
Simulates async SQLite queries using localStorage. Swap the body of dbQuery for a real expo-sqlite transaction and everything else stays identical.
State + render (state, render, applyFilters)
A plain object holds { loading, error, allProducts, products, category, search, sort, wishlist, cartCount, addingId }. Every user action mutates state and calls render(), which rebuilds the grid.
Event handlers (handleAdd, handleWish, handleSort, loadProducts, …)
Async functions that call the DB layer, update state, and call render().

src/ — React Native–ready modules
These files mirror the index.html logic exactly, but as proper ES modules with import/export.
src/database/db.js
Exposes: getProducts(category), getProductById(id), getCategories(), getCart(), addToCart(id), removeFromCart(id), getWishlist(), toggleWishlist(id), resetDatabase().
To migrate to real SQLite, replace only the _query() function body:
jsimport * as SQLite from 'expo-sqlite';
const db = SQLite.openDatabase('maison.db');

db.transaction(tx => {
  tx.executeSql(query, params,
    (_, { rows }) => resolve(rows._array),
    (_, error)    => reject(error)
  );
});
src/hooks/useProducts.js
Exports: useProducts(category), useCategories(), useCart(), useWishlist().
Each hook returns { data, loading, error, refetch } — same pattern as React Query / SWR.
src/components/ProductCard.jsx
Renders a single product card as a tagged template string (works in vanilla JS; trivially convertible to JSX for React Native).

Migrating to React Native / Expo

Copy src/database/db.js → replace localStorage with expo-sqlite
Copy src/hooks/useProducts.js → works as-is, zero changes
Rebuild UI screens using FlatList, View, Text, Pressable
The data layer and all business logic require no changes


Seed Data
12 products across 4 categories: Electronics, Fashion, Home, Beauty.
Data is seeded into localStorage on first load under the key maison_db_v2.
To reset: open DevTools → Application → Local Storage → delete maison_db_v2, then refresh.

Known Intentional Behaviours

4% error rate on getProducts — simulates a flaky DB connection. The error state + retry button is meant to demonstrate error handling. Adjust errorRate in db.js / the inline dbQuery call to change it.
Simulated latency (600–1100 ms) — skeleton cards cover this window. Reduce minMs/maxMs in dbQuery for faster dev iteration.
No real backend — all data lives in the browser's localStorage. Clearing site data resets the app to seed state.


Tech Stack
LayerChoiceWhyUIVanilla HTML/CSS/JSZero setup, runs from a file openFontsGoogle Fonts (Playfair Display + DM Sans)Loaded via <link>DatalocalStorage (JSON)Mirrors SQLite API, trivially swappableBuildNoneLearning-friendly; production would use Vite or Metro

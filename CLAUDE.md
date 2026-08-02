# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Marketing/landing page for Intermezzo, an Italian restaurant in Brussels. Single-page site entirely contained in `index.html` (inline `<style>` and `<script>`, no external JS/CSS files). No build step, package manager, or test suite.

## Commands

- **Run**: open `index.html` directly in a browser. No server or build required.
- There is no lint, test, or build command — none are configured.

## Architecture

Everything lives in `index.html`:

- **i18n**: the `I18N` object holds full FR/NL/EN translations keyed by string id (`nav_home`, `hero_tagline`, etc., including per-language menu `categories`). `applyLanguage(lang)` swaps `document.title`, the meta description, every `[data-i18n]` element (via `setHTML`, which uses `innerHTML` only when the value contains `<` or `&`), and re-renders the menu/hours. Language choice persists in `localStorage` (`intermezzo_lang`). **When adding any user-facing string, add a matching key to all three language blocks (fr/nl/en) and reference it with `data-i18n` rather than hardcoding text.**
- **Menu data**: `MENU_ITEMS` (dish names + prices, language-independent since dish names are Italian) and `CATEGORY_ORDER` drive `renderMenu`, which builds the tab buttons and panels for each category. Category *labels/notes* are per-language and come from `I18N[lang].categories`, not from `MENU_ITEMS`.
- **Hours & live status**: `HOURS_ROWS` (display strings per language via `t.days`) and `SCHEDULE` (minute-of-day open/close ranges per weekday, Brussels time) are two parallel representations of the same opening hours — both must be updated together when hours change. `getBrusselsParts()` uses `Intl.DateTimeFormat` with `timeZone: "Europe/Brussels"` to get the current weekday/time regardless of the visitor's own timezone; `updateStatus()` polls this every 60s to toggle the open/closed badge.
- **Reservation**: the "Réserver" button links to `wa.me/<WHATSAPP_NUMBER>` with a prefilled, per-language message — not a form or backend.
- **Nav/scroll behavior**: an `IntersectionObserver` over `section[id]` elements highlights the active nav link; header background/shadow toggles via a scroll listener; mobile nav is a slide-in panel toggled by `.open`.

Practical implication: because there's no backend, all content changes (menu items, prices, hours, contact info) are made directly in the `MENU_ITEMS`, `HOURS_ROWS`/`SCHEDULE`, and `I18N` objects in this file.

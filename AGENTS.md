# Agent guide: Free Shipping Bar for Shopify

Instructions for AI coding agents (Claude Code, Cursor, Codex, etc.) installing or customizing this section. Humans should start with [README.md](README.md).

## What this is

A single Shopify theme section, `sections/free-shipping-bar.liquid`. It puts a progress bar with up to 4 reward steps inside the theme's **cart drawer**. It updates live as the cart changes, adds and removes a free gift product, and shows "You're saving $X" at the cart total. No app.

```
sections/free-shipping-bar.liquid   The product. Markup, CSS, JS and schema in one file.
README.md                           Install steps, discount setup, settings, troubleshooting
```

The section only does display and the gift line. **The discounts themselves are Shopify automatic discounts** the merchant creates in admin (README, Part 2). Nothing in this repo can create them: Shopify CLI has no discount commands, and creating them from storefront JS would need an Admin API token, which must never ship in a theme.

## Installing into a theme

With Shopify CLI, from the user's theme directory:

1. Copy `sections/free-shipping-bar.liquid` into the theme's `sections/` folder.
2. Add it to the **footer group** (it renders nothing visible there; it mounts itself into the cart drawer). Pull the file first so you don't overwrite the merchant's footer edits:
   ```bash
   shopify theme pull --only sections/footer-group.json
   ```
   Then add under `sections`, and append the key to `order`:
   ```json
   "free_shipping_bar": { "type": "free-shipping-bar", "settings": { "gift_product": "<gift-product-handle>" } }
   ```
   Empty `settings` uses the schema defaults ($65 / $100 / $150 / $185, no gift).
3. Make sure the theme's cart type is **drawer** (Horizon and Dawn: `settings.cart_type == "drawer"` in `config/settings_data.json`).
4. Push to an **unpublished** theme first: `shopify theme push --unpublished --theme "Free shipping bar"`, or `--theme <id> --only sections/free-shipping-bar.liquid --only sections/footer-group.json`.
5. Ask the human before pushing to the live theme (`--allow-live`).
6. Remind the human to create the automatic discounts (README, Part 2) with the **same amounts** as the section settings. If you have browser access to their admin and they ask you to, you can create them there. Ask before creating the "All products" collection, since it also appears on the storefront.

## Hard rules when changing the section

These keep it portable. Breaking one usually works in the theme you're testing and fails in every other theme.

1. **Stay one file.** CSS goes in `{% stylesheet %}`, JS in `{% javascript %}`. No new files in `assets/` or `snippets/`. Neither block can contain Liquid; all settings reach the JS through the JSON in `<script type="application/json" data-fsb-config>`.
2. **No theme dependencies for core behavior.** Theme-specific selectors are allowed only as defaults in `DEFAULTS` or as optional extras that do nothing when they don't match.
3. **Prefix everything.** Classes `fsb-*` / `fsb__*`, CSS variables `--fsb-*`, custom element `<free-shipping-bar>`, cart line property `_fsb_gift`, events `fsb:*`.
4. **Every shopper-facing string is a setting.** Schema labels are plain English, not `t:` keys, because the section can't ship locale files.
5. **Guard the element definition**: `if (customElements.get('free-shipping-bar')) return;`.
6. **Progress counts the cart before discounts**, excluding the gift line. Don't switch it to `total_price`: the bar would flicker as discounts push the cart back under a threshold.
7. **Keep Liquid out of `{% javascript %}`**, including literal `{{ }}` in strings. The money-format fallback is built with string concatenation on purpose.

## How it works

### Liquid (top of the file)
Builds one JSON config: style, money format, the 4 steps (`amount`, `label`, `reward`, `icon`, `isGift`), the gift product (variant ID, handle, title, image, available), all strings, colors and the 4 advanced selectors. Steps with amount `0` are dropped in JS.

### Mounting
`<free-shipping-bar>` stays hidden in the footer. On connect it builds a separate `.fsb` element and inserts it into the cart drawer: first match of the **drawer selector**, then before the first match of the **mount selector** inside it. A `MutationObserver` on `document.body` re-inserts it and re-applies decorations whenever the theme redraws the drawer. Horizon's morph removes unknown nodes, so this matters.

### Knowing when the cart changed
Three sources, all funneled into a debounced `refresh()` that fetches `/cart.js`:
- `shopify:cart:lines-update`, Shopify's standard cart event. Horizon and newer themes dispatch it.
- A one-time `window.fetch` wrapper that fires `fsb:cart-changed` after any request to `/cart/add|change|update|clear`. This covers themes and apps that don't use the standard event. The bar's own requests send an `X-FSB` header and are skipped.
- `cart:updated`, a name some themes use.

### The free gift (`syncGift`)
- Unlocked (progress at or above the gift step) and no gift line: add 1 of the gift variant with line property `_fsb_gift: 1`. Underscore properties are hidden at checkout by Shopify.
- Locked and a gift line exists: remove it.
- Gift quantity above 1: set it back to 1.
- If the shopper removes the gift themselves while it's unlocked, it isn't re-added for the rest of the session (`sessionStorage` key `fsb-gift-declined`, cleared when they drop below the threshold).
- After every gift change, `notifyTheme()` asks the theme to redraw. It dispatches Shopify's `CartLinesUpdateEvent` (imported from `https://cdn.shopify.com/storefront/standard-events.js`, the same module Horizon uses) with `sections` HTML for every `cart-items-component[data-section-id]`, plus a generic `cart:refresh` event.

### Decorating the theme's cart (`decorate`)
Best effort, re-applied after every redraw:
- **Gift row:** found by `[data-key="<line key>"]`, else by a link to `/products/<gift handle>` and `closest(row selector)`. Gets class `fsb-gift-row`, a `.fsb-badge`, and every outermost `[class*="quantity"]` element inside it gets `visibility: hidden`.
- **Total:** the first selector in the **total selector** list that matches (tried in order, not as one selector list). Gets a `<s class="fsb-was">` original total and a `.fsb-saving` line after its row. If nothing matches, the savings show under the bar instead (`.fsb__savefallback`).

### Confetti
`celebrate()` compares the previous and new progress. Only the highest step crossed in one change celebrates. Last step: `shower()` across the drawer. Others: `burst()` from that step's icon. Skipped when `prefers-reduced-motion` is set or the setting is off.

### Money
`money()` formats cents with the shop's `money_format` (all Shopify `{{ amount* }}` variants). Thresholds are multiplied by `Shopify.currency.rate` so they hold in other presentment currencies.

## Supporting another theme

Tested on **Horizon**. Defaults in `DEFAULTS` also cover common Dawn-family names. When something doesn't line up, ask for a screenshot of the open cart drawer, then:

| Symptom | Where to fix |
|---|---|
| Bar missing from the drawer | Find the drawer's root element. Set it in the **Cart drawer selector** setting, or add it to `DEFAULTS.drawer` |
| Bar in the wrong spot | **Insert the bar before this element** setting / `DEFAULTS.mount` |
| Gift row has no badge or still shows quantity | **Cart line row selector** / `DEFAULTS.row` |
| No savings at the total | **Cart total value selector** / `DEFAULTS.total` (list the most specific selector first) |
| Gift added but drawer doesn't show it until reopened | The theme doesn't listen for `shopify:cart:lines-update`. Add the theme's own refresh call at the end of `notifyTheme()`, guarded so it does nothing in other themes. For Dawn, its `cart-drawer` element has a `renderContents()` method that takes a cart response with `sections` for `cart-drawer` and `cart-icon-bubble`; request those section IDs in `cartRequest()` and pass the response to it |

Settings are the first choice for a single store. Edit `DEFAULTS` when you're improving the section for everyone.

## Checking your work

There are no automated tests. Verify on a real storefront (preview of an unpublished theme):

1. Empty cart, add one cheap item, open the drawer: the message names the first reward and the right remaining amount.
2. Change quantities inside the drawer to cross each step: the bar moves without a reload, each step pops confetti.
3. Cross the gift step: the gift is added at $0 (needs the FREE GIFT discount), the theme row gets the badge, no quantity box, big confetti.
4. Drop below the gift step: the gift is removed and the locked row returns.
5. With discounts active: crossed-out original and "You're saving" at the cart total, and the numbers match checkout.
6. Check phone width (under 750px).
7. In the theme editor, change a setting: the bar rebuilds with no duplicate bars in the drawer.

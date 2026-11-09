# Free Shipping Progress Bar for Shopify

A progress bar for your cart drawer that shows shoppers how close they are to free shipping, a discount and a free gift. It fills as they add to cart, celebrates each reward with confetti, and drops the free gift into the cart on its own.

It's one file. No app, no monthly fee, no code needed to install it.

**Using Claude Code, Cursor or another AI coding tool?** Point it at this repo and say what you want, like "install this in my theme" or "change the gift to 2 items". [AGENTS.md](AGENTS.md) tells it how the section works and how to change it without breaking it in other themes.

## What it does

Out of the box it has four steps. Every amount, label and message is a setting you can change.

| Cart total | Shopper gets |
|---|---|
| $65 | Free shipping |
| $100 | 10% off |
| $150 | 20% off |
| $185 | A free gift, added to the cart automatically |

- **Live updates.** The bar moves as soon as the cart changes: add to cart, change a quantity, remove an item. No page reload.
- **A message that tells them what's next**, like "Add $33 more to get free shipping!"
- **Confetti** when a step unlocks. A small burst for each step, a full shower for the last one. Shoppers who have reduced motion set on their device don't see it.
- **A locked gift preview.** Before $185, the gift shows grayed out with a lock and "Add $42 more to unlock". At $185 it unlocks, goes into the cart at $0 and gets a FREE GIFT badge. If the cart drops below $185, the gift comes back out.
- **"You're saving $X"** next to the cart total, with the original price crossed out.
- **Two styles:** one bar with an icon at each step, or four separate segments. Pick one in the theme editor.
- **Counts the cart before discounts**, the same way Shopify checks discount minimums. Otherwise 20% off could knock a $160 cart back under $150.

Built and tested on Shopify's **Horizon** theme. It's written to work in other Online Store 2.0 themes too (every free Shopify theme since 2021). See [Other themes](#other-themes) if something doesn't line up.

## How it works

The bar does two jobs: it shows progress and it adds or removes the free gift. **Shopify does the actual discounting** with automatic discounts that you set up once in your admin. The shopper never types a code. Your checkout shows each discount by name, like "20% OFF (-$11.59)" and "FREE GIFT (-$68.00)".

So setup has two parts: add the section (Part 1), then create the discounts (Part 2). Both take about 10 minutes total.

## Part 1: Add the section (about 3 minutes, no coding)

**Tip:** try it on a copy of your theme first. In **Online Store > Themes**, click **...** next to your theme and choose **Duplicate**. Test on the copy, then publish it.

### Step 1: Add the section file

1. Open [`sections/free-shipping-bar.liquid`](sections/free-shipping-bar.liquid) here on GitHub and click the **Copy raw file** button (the two-squares icon at the top right of the file).
2. In your Shopify admin, go to **Online Store > Themes**. Next to your theme, click **...** then **Edit code**.
3. In the file list on the left, find the **sections** folder and click **Add a new section**.
4. Name it `free-shipping-bar` and pick **Liquid** if it asks.
5. Delete the starter code Shopify puts in the new file, paste in what you copied, and click **Save**.

### Step 2: Turn it on in the theme editor

1. Make sure your theme's cart opens as a **drawer**. In most themes that's **Theme settings > Cart > Cart type: Drawer**.
2. Go to **Online Store > Themes > Customize**.
3. In the left sidebar, scroll to the **Footer** area and click **Add section**. Pick **Free shipping bar**.
   It doesn't show anything in the footer. It places itself inside the cart drawer.
4. In the section settings, pick your **Gift product** under **Free gift**.
5. Click **Save**, add something to the cart on your storefront and open the cart.

### Using Shopify CLI instead

Copy `sections/free-shipping-bar.liquid` into your theme's `sections/` folder, add it to `sections/footer-group.json`, and push to an unpublished theme:

```json
"free_shipping_bar": { "type": "free-shipping-bar", "settings": { "gift_product": "your-gift-product-handle" } }
```

Add `"free_shipping_bar"` to the end of the `order` list in the same file, then:

```bash
shopify theme push --unpublished --theme "Free shipping bar test"
```

## Part 2: Create the discounts in Shopify (about 7 minutes)

Shopify CLI can't create discounts, so this part is in your Shopify admin. An AI agent with browser access can click through it for you too.

### Step 1: Create an "All products" collection

The free gift discount needs a collection that holds every product. Most stores don't have one.

Go to **Products > Collections > Create collection**. Title it `All products` and add one condition: **Price is greater than 0**. Save. It picks up every product automatically, including ones you add later.

### Step 2: Create four automatic discounts

Go to **Discounts > Create discount** and make each of these. For every one, set **Method** to **Automatic discount**. The title is what shoppers see at checkout.

| Title | Discount type | Settings | Combinations (check these) |
|---|---|---|---|
| FREE SHIPPING | Free shipping | Minimum purchase amount: $65 | Product discounts, Order discounts |
| 10% OFF | Amount off order | Percentage: 10. Minimum purchase amount: $100 | Product discounts, Shipping discounts |
| 20% OFF | Amount off order | Percentage: 20. Minimum purchase amount: $150 | Product discounts, Shipping discounts |
| FREE GIFT | Buy X get Y | Customer spends: Minimum purchase amount $185, any items from **All products**. Customer gets: quantity 1 of your gift product, at a discounted value of **Free**. Check **Set a maximum number of uses per order**: 1 | Order discounts, Shipping discounts |

Two things that matter:

- **Leave "Order discounts" unchecked on 10% OFF and 20% OFF.** That stops them from stacking into 30%. Shopify gives the shopper whichever is bigger.
- **Keep the amounts in sync.** If you change a step to $75 in the section settings, change the matching discount to $75 too. The bar and checkout don't talk to each other.

### Step 3: Test it

Add items until you pass $185. You should see each step light up, the gift drop into the cart at $0.00, and "You're saving $X" under the cart total. If a cart was already open before you created the discounts, change a quantity once so Shopify recalculates it.

## Settings

Everything below is in the theme editor under **Footer > Free shipping bar**.

| Setting | What it does |
|---|---|
| Bar style | **Milestone track**: one bar with an icon at each step. **Step segments**: one short bar per step with its dollar amount. |
| Step 1 to 4: Unlocks at | The cart total for that step, in your store's currency. Set to 0 to turn a step off. You can run anywhere from 1 to 4 steps. |
| Step 1 to 4: Label | The word under the bar, like "Free shipping" |
| Step 1 to 4: Reward in the message | What fills in `[reward]` in the progress message, like "free shipping" or "a free gift" |
| Step 1 to 4: Icon | Truck, percent, gift, star or heart |
| Gift product | The product the bar adds to the cart. Leave it empty to turn the gift off. |
| Gift unlocks at | Which step adds the gift. Step 4 by default. |
| Locked gift message | Shown on the grayed-out gift. `[amount]` is how much more they need. |
| Unlocked gift message | Shown for a moment when the gift unlocks |
| Gift badge | The label on the gift in the cart. "FREE GIFT" by default. |
| Progress message | `Add [amount] more to get [reward]!` by default |
| All steps unlocked | Shown when every step is reached |
| Show savings at the cart total | Adds the crossed-out original total and the savings line |
| Savings message | `You're saving [amount]` by default |
| Show bar when the cart is empty | Off by default |
| Confetti when a step unlocks | On by default |
| Colors | Bar fill (red by default), bar background, unlocked checkmarks, panel background, text |
| Theme compatibility (advanced) | Leave these blank. See [Other themes](#other-themes). |

Every word a shopper sees is a setting, so you can translate it or change the tone without touching code.

## Other themes

The bar finds your theme's cart drawer, the line items and the cart total on its own. It knows Horizon's markup and the common names used in Dawn and most other themes. If your theme names things differently, one of these will happen:

| What you see | Fix |
|---|---|
| The bar doesn't show up in the cart drawer | Fill in **Cart drawer selector** with your drawer's element, like `#CartDrawer` |
| The bar shows up in the wrong spot | Fill in **Insert the bar before this element** with the element it should sit above, like `.drawer__contents` |
| The gift is in the cart but has no FREE GIFT badge, or still has a quantity box | Fill in **Cart line row selector** with the element that wraps one cart line |
| No "You're saving" line under the total | Fill in **Cart total value selector** with the element that holds the total amount. If none is found, the savings line shows under the bar instead. |
| The gift is added but doesn't appear until you close and reopen the cart | Your theme doesn't listen for Shopify's standard cart events. An AI agent can add your theme's refresh call. See [AGENTS.md](AGENTS.md). |

To find a selector: right-click the element on your storefront, choose **Inspect**, and copy its `id` (put `#` in front) or one of its classes (put `.` in front). Or skip that and ask your AI agent with a screenshot of the cart drawer.

The gift and the discounts still work even if the badge or savings line doesn't show. The shopper sees "FREE GIFT" and the $0 price at checkout either way.

## Customize it with Claude

The code is commented and lives in one file. You can change it with [Claude Code](https://claude.com/claude-code), or paste the file into Claude and paste the answer back into **Edit code**. Prompts that work:

**Change the rewards**
> Change the steps to free shipping at $50, 15% off at $120 and a free gift at $200. Update the defaults in the schema and the README discount table to match.

> Make the gift step let the shopper choose between two gift products.

**Change the look**
> Put the bar above the "Cart" title in the drawer, with the close button on the same row.

> Use my theme's heading font for the progress message.

**Change the behavior**
> Count items instead of dollars: free shipping at 3 items, 10% off at 6.

> Turn off the confetti on phones but keep it on desktop.

**Fit a different theme**
> I installed free-shipping-bar on the [theme name] theme. The free gift gets added but the cart drawer doesn't show it until I reopen it. Make the bar refresh my theme's cart drawer after it adds the gift.

Tell Claude which theme you're on and exactly what you see. A screenshot helps.

## License

MIT. Use it on your store or your clients' stores, change it, ship it.

# CookTrace shopping list

If you also run [CookTrace](../cooktrace/index.md), NoteTrace works with your CookTrace [shopping list](../cooktrace/shopping.md): see it in a **Shopping** page, grouped by aisle, check things off as you shop, and send a NoteTrace checklist's open items to it. CookTrace stays the one shopping list; NoteTrace shows it live and adds to it, it doesn't keep a second copy.

## Set it up

**In CookTrace**, sign in as the account whose shopping list you use, open **Settings, API Tokens**, and create a token with just the **shopping** scope. That token can list, add to, check off, and clear your shopping list, and reach nothing else. Copy it (`ct_pat_...`); it's shown once. CookTrace needs no other setting.

**In NoteTrace**, open **Settings, CookTrace** (under Integrations) and turn on **Enable CookTrace**. Enter the **Base URL** (for example `https://cook.example.com`) and paste the token into **API Token**. Each field saves when you leave it: NoteTrace checks the address and token, and the bar at the top of the card says **Connected** with your CookTrace username, or what's wrong: a wrong token, a token without the shopping scope, or a CookTrace too old to have the shopping list API (update CookTrace). **Test** checks the saved link again at any time.

Once saved, the token shows as dots. NoteTrace never shows it again; paste a new token to replace it. If you change the address, paste the token again too: the saved token is only ever sent to the address it was linked with.

Each NoteTrace account links its own CookTrace account.

### CookTrace on your local network

NoteTrace refuses private and loopback addresses by default, so a link can't be used to probe your network. If CookTrace runs on your LAN (`http://192.168.1.20:3003`), on the same host (`http://localhost:3003`), or on the same Docker network (`http://cooktrace:3001`), set this on the NoteTrace server:

```yaml
services:
  notetrace:
    environment:
      - ALLOW_PRIVATE_COOKTRACE_URLS=1
```

Cloud metadata addresses stay blocked either way.

## The Shopping page

Once CookTrace is linked and on, **Shopping** appears in the menu under Tasks.

- Items are grouped by aisle in CookTrace's order, with items that have no aisle under **Other**. Each shows its amount and, when it came from a recipe, the recipe's name.
- Tap the circle to check an item off. It moves to **checked**, a collapsed group at the bottom, and is checked off in CookTrace too. Tap it there to put it back.
- **Add to the shopping list** adds an item. CookTrace fills in its aisle when the name matches a pantry item, and doesn't add it twice when it's already on the list.
- **Clear Checked** removes checked items from the CookTrace list.
- The list reloads when you open the page, come back to the app, or tap refresh. **Open in CookTrace** (the arrow at the top) opens CookTrace's own shopping page for editing amounts, aisles, and order.

**Without a connection** the page shows the last list it loaded, with when that was. Checks, adds, and clears still work: they show at once, a note says how many changes are waiting, and they go to CookTrace, in order, when the connection is back.

## Send a list

Open a checklist and tap **Send to CookTrace** (the cart icon in the editor bar; it appears on checklists once CookTrace is linked and on). NoteTrace lists the unchecked items it will send. Tick **Check Off Sent Items** to check them off in the note afterwards, then tap **Send 3 Items** (the count matches your list).

- Only unchecked items are sent. Duplicate items (ignoring case) are sent once.
- Formatting and `[[link]]` brackets are removed, so `**Flour**` arrives as "Flour".
- Up to 50 items go in one send.
- CookTrace capitalizes names as it does for items you add there, gives an item the aisle of a pantry item with the same name, and skips one that's already on the list unchecked. NoteTrace says how many were added and how many were already there.

## How it works

The CookTrace token is stored encrypted on the NoteTrace server and is never sent to your browser or phone. Your device asks the NoteTrace server, and the server calls CookTrace's shopping list API (`/api/v1/shopping`) with your token. CookTrace's per-token rate limit (60 requests a minute by default) applies.

Linking needs a NoteTrace server, so it isn't available in Android local mode. Turning off **Enable CookTrace** hides Shopping and Send to CookTrace and keeps the link. To remove it, tap **Unlink** under **Remove Link** in Settings, CookTrace, and revoke the token in CookTrace.

## Related

- [CookTrace shopping list](../cooktrace/shopping.md)
- [Federation (cross-app links)](../integrations/federation.md)

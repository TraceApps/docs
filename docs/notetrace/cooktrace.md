# Send to CookTrace

If you also run [CookTrace](../cooktrace/index.md), a NoteTrace checklist can send its open items to your CookTrace [shopping list](../cooktrace/shopping.md). Keep writing the list wherever it's quickest, then send it when you head to the store.

## Set it up

**On the CookTrace server**, turn on MCP writes and restart it:

```yaml
services:
  cooktrace:
    environment:
      - MCP_ENABLED=1
      - MCP_WRITE_ENABLED=1
```

**In CookTrace**, sign in as the account whose shopping list you use, open **Settings, API Tokens**, and create a token with the `mcp:write` scope. Copy the token (`ct_pat_...`); it's shown once.

**In NoteTrace**, open **Settings, CookTrace** (under Integrations), enter the CookTrace address (for example `https://cook.example.com`) and the token, and tap **Link CookTrace**. NoteTrace checks both before saving and says what's wrong if something is missing: a wrong token, a token without `mcp:write`, or MCP writes turned off on CookTrace.

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

## Send a list

Open a checklist and tap **Send to CookTrace** (the cart icon in the editor bar; it appears on checklists once CookTrace is linked). NoteTrace lists the unchecked items it will send. Tick **Check Off Sent Items** to check them off in the note afterwards, then tap **Send 3 Items** (the count matches your list).

- Only unchecked items are sent. Duplicate items (ignoring case) are sent once.
- Formatting and `[[link]]` brackets are removed, so `**Flour**` arrives as "Flour".
- Up to 50 items go in one send.
- Items are added as new rows on the CookTrace shopping list, where you can edit, group, and check them off like anything else. CookTrace capitalizes item names as it does for items you add there.
- If CookTrace refuses an item partway through, NoteTrace stops and tells you how many were added.

## How it works

The CookTrace token is stored encrypted on the NoteTrace server and is never sent to your browser or phone. When you send, your device asks the NoteTrace server, and the server calls CookTrace's MCP endpoint (`add_shopping_item`, once per item) with your token. Nothing needs to change on CookTrace beyond turning on MCP writes. CookTrace's per-token rate limit (60 requests a minute by default) applies.

Linking needs a NoteTrace server, so it isn't available in Android local mode. To stop, tap **Unlink** in Settings, CookTrace, and revoke the token in CookTrace.

## Related

- [CookTrace shopping list](../cooktrace/shopping.md)
- [CookTrace MCP](../cooktrace/mcp.md)
- [Federation (cross-app links)](../integrations/federation.md)

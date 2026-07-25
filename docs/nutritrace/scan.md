# Barcode and Scan Label

Two camera-driven ways to get a food into NutriTrace without typing it out. **Barcode** reads a product code and looks it up. **Scan Label** photographs a nutrition-facts panel and lets your configured AI provider fill in every field.

## Barcode scanner

On Android the scanner uses `@capacitor-mlkit/barcode-scanning` (Google's ML Kit), with Google Code Scanner as a fallback when ML Kit is not available on the device. On the web PWA the browser's built-in Barcode Detection API handles it where present, and QuaggaJS fills in for browsers that lack native support. Formats recognized: UPC-A, UPC-E, EAN-8, EAN-13, and QR codes (for future federation share links).

Tap the barcode icon in the Foods picker or in the Diary add sheet, point the camera at the product, and the app matches on the first clean read. Toggles under **Settings → Foods**:

- `barcodeBeep`: audio confirmation on match. On by default.
- `barcodeFlashlight`: turns the camera flashlight on when the scanner opens. Device-only (`DEVICE_PREFS`), because there is no flashlight on desktop and each phone chooses independently.

### Match, then what

When the barcode is **in Open Food Facts** (the common case), the product opens straight into the quantity sheet with name, brand, and per-serving nutrition prefilled from OFF. Log it, done.

When the barcode is **not in OFF**, the food editor opens **prefilled with the barcode number**. Fill in the name, brand, serving size, and macros yourself. If you have OFF account credentials configured under **Settings → Connected Services → Open Food Facts**, a **Share to Open Food Facts** button uploads your entry (with the product photo) to the community database. Next time anyone scans that barcode, OFF returns your data. This is the primary way to make OFF better without leaving NutriTrace.

Barcodes not in OFF are also fine to keep private: save the food, skip the share, and it stays in your Local catalog with the barcode attached so your own next scan hits it.

## Scan Label (AI OCR)

Barcodes only help for packaged products with a scannable code. For everything else (a restaurant menu card, a piece of home baking with a printed nutrition sticker, a foreign product where the barcode is not in OFF), **Scan Label** takes a photo of the actual Nutrition Facts panel and asks your configured AI provider to extract every value.

The button lives on the **Nutrition card** inside the food editor. Tap it, snap the label, wait a second or two. Calories, protein, carbs, fat, saturated fat, fiber, sugar, sodium, cholesterol, potassium, calcium, iron, and the other micros NutriTrace tracks all fill in, along with **serving size**. Review, adjust if the OCR misread anything, save.

### What providers work

Scan Label sends a base64 image over the AI chat endpoint (`POST /api/ai/chat`), which is capped at 12 MB per request (`server/index.js:76`). The provider has to support vision:

- **Claude**: any of the vision-capable models.
- **OpenAI**: `gpt-4o` and successors.
- **Gemini**: any of Google's multimodal models.
- **OpenAI-compatible endpoints**: any multimodal model your endpoint serves (Ollama with Llama-vision variants, Qwen-VL through vLLM, LM Studio with a vision model loaded, and so on).

If your configured model does not support vision, the Scan Label button hides. Switch to a vision-capable model under **Settings → AI Assistant** and it comes back.

### Cost and privacy notes

One scan is one API call. Costs are trivial on the cloud tiers (fractions of a cent per label on Claude / OpenAI / Gemini). On a local model with vision, the cost is your electricity. The photo is sent as-is to the provider you configured; it is not stored server-side beyond the chat history record. If you would prefer no cloud roundtrip at all, run a local vision model through the OpenAI-compatible slot.

## Related

- [Foods, meals and recipes](foods.md)
- [Open Food Facts](off.md)
- [Trace in NutriTrace + Smart Log](trace.md)

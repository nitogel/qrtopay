# QR to Pay

A single-page static web app that generates Ukrainian NBU-standard QR codes for credit transfers (кредитові перекази).

## Usage

Open `index.html` directly in a browser, or serve it locally:

```bash
python3 -m http.server 8080
# or
npx serve .
```

Fill in the form fields and click **Згенерувати QR-код**:

| Field | Description |
|---|---|
| IBAN | Recipient's bank account (UA...) |
| ЄДРПОУ / РНОКПП | Recipient's tax ID or passport number |
| Отримувач | Recipient name (person or organization) |
| Сума (UAH) | Transfer amount |
| Призначення платежу | Payment purpose (max 140 characters) |

The generated QR code encodes a `https://bank.gov.ua/qr/<payload>` URL that any Ukrainian banking app can scan to pre-fill a transfer.

## QR Payload Format

The payload follows the [NBU specification](NBU.md) — 13 newline-separated fields, Base64url-encoded:

```
BCD
001
1
UCT
              ← BIC (reserved, empty)
{recipient}
{IBAN}
UAH{amount}   ← e.g. "UAH576.45" or "UAH3"
{edrpo}
              ← Ціль (reserved, empty)
              ← Reference (reserved, empty)
{purpose}
              ← Display text (reserved, empty)
```

Constraints:
- Total payload ≤ **331 bytes** (UTF-8)
- Error correction level: **M**
- Max QR version: **13** (69×69 modules)

## Dependencies

- [Tailwind CSS](https://tailwindcss.com/) 2.2.19 — loaded from CDN
- [QRCode.js](https://github.com/soldair/node-qrcode) 1.0.0 — bundled as `qrcode.bundle.js`

## Specification

See [NBU.md](NBU.md) for the full National Bank of Ukraine regulation on QR code formation and usage for credit transfers.
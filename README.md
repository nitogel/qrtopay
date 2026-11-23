# QR to Pay

A single-page static web app that generates Ukrainian NBU-standard QR codes for credit transfers (кредитові перекази).

**Live version: [nitogel.github.io/qrtopay](https://nitogel.github.io/qrtopay/)**

## Usage

Open `index.html` directly in a browser, or serve it locally:

```bash
python3 -m http.server 8080
# or
npx serve .
```

Fill in the form fields — the QR code is generated automatically:

| Field | Description |
|---|---|
| IBAN | Recipient's bank account (UA..., exactly 29 characters) |
| ЄДРПОУ / РНОКПП | Recipient's tax ID (max 10 characters) |
| Отримувач | Recipient name (max 140 characters) |
| Сума (UAH) | Transfer amount (max 999999999.99) |
| Призначення платежу | Payment purpose (max 420 characters) |

The generated QR code encodes a `https://bank.gov.ua/qr/<payload>` URL that any Ukrainian banking app can scan to pre-fill a transfer.

## Pre-filling via URL Query Parameters

All form fields can be pre-filled by passing query parameters:

| Parameter | Field |
|---|---|
| `iban` | IBAN |
| `edrpo` | ЄДРПОУ / РНОКПП |
| `recipient` | Отримувач |
| `amount` | Сума |
| `purpose` | Призначення платежу |

Example:

```
index.html?iban=UA213223130000026007233566001&recipient=Іван%20Петренко&amount=150&purpose=Оплата%20послуг&edrpo=12345678
```

Only the provided parameters are applied; omitted ones leave their inputs empty.

## QR Payload Format

The payload follows format version **002** of the [NBU specification](https://zakon.rada.gov.ua/laws/show/v0097500-25#n16) (Resolution № 97 of 19.08.2025, annex 3). The QR data is a hyperlink: `https://bank.gov.ua/qr/` + Base64URL-encoded open data structure of 13 newline-separated fields:

```
BCD
002
1             ← encoding (1 = UTF-8)
UCT
              ← BIC (reserved, empty)
{recipient}
{IBAN}
UAH{amount}   ← e.g. "UAH576.45" or "UAH3" (whole → no decimals, otherwise exactly two)
{edrpo}
              ← Ціль (reserved, empty)
              ← Reference (reserved, empty)
{purpose}
              ← Display text (reserved, empty)
```

Constraints:
- Base64URL-encoded part ≤ **475 bytes** (total hyperlink ≤ 507)
- Error correction level: **M** (level L is forbidden for format 002)
- QR version: **10–17**
- The hryvnia sign (₴) in a white circle at the QR center is mandatory; circle diameter depends on the QR version (annex 1, §§ 10–13)

## Dependencies

- [Tailwind CSS](https://tailwindcss.com/) 2.2.19 — only the used utility classes, inlined in `index.html`
- [node-qrcode](https://github.com/soldair/node-qrcode) — bundled as `qrcode.bundle.js` (`npm run build`)

## Specification

[Постанова Правління НБУ № 97 від 19.08.2025](https://zakon.rada.gov.ua/laws/show/v0097500-25#n16) — Правила формування, передачі та обробки структури даних і графічного зображення QR-коду для обміну реквізитами кредитових та миттєвих кредитових переказів (чинна з 01.11.2025, замінює постанову № 68 від 28.05.2020).

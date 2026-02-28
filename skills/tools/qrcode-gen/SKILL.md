---
name: qrcode-gen
description: Generate QR codes from text, URLs, or data. Quick CLI tool for creating scannable QR codes as PNG or terminal output.
version: 1.0.1
metadata:
  openclaw:
    emoji: "📱"
    requires:
      bins: ["qrencode"]
    install:
      - id: apt
        kind: apt
        packages: ["qrencode"]
        bins: ["qrencode"]
        label: "Install qrencode (apt)"
      - id: brew
        kind: brew
        formula: "qrencode"
        bins: ["qrencode"]
        label: "Install qrencode (brew)"
---

# QR Code Generator

Generate QR codes instantly from any text or URL.

## Usage

### Terminal Output (ASCII)
```bash
qrencode -t ANSI "https://example.com"
```

### Save as PNG
```bash
qrencode -o code.png "Your text here"
qrencode -o code.png -s 10 "https://skillscan.dev"  # larger size
```

### With custom size and margin
```bash
qrencode -o code.png -s 8 -m 2 "Hello World"
# -s = module size (pixels)
# -m = margin (modules)
```

### WiFi QR Code
```bash
qrencode -o wifi.png "WIFI:T:WPA;S:NetworkName;P:Password;;"
```

### vCard Contact
```bash
qrencode -o contact.png "BEGIN:VCARD
VERSION:3.0
N:Doe;John
TEL:+1234567890
EMAIL:john@example.com
END:VCARD"
```

## Examples

| Use Case | Command |
|----------|---------|
| URL | `qrencode -t ANSI "https://github.com"` |
| Text | `qrencode -o msg.png "Secret message"` |
| Email | `qrencode -t ANSI "mailto:hi@example.com"` |
| Phone | `qrencode -t ANSI "tel:+1234567890"` |

## Tips

- Use `-t ANSI` for quick terminal preview
- Use `-t PNG` (default) for image files
- Larger `-s` values = bigger, easier to scan
- Test QR codes with your phone camera before sharing

---
*Last updated: 2026-02-02*

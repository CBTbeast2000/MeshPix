# Meshpix

Send tiny pictures over [MeshCore](https://github.com/meshcore-dev/MeshCore) as ordinary text messages. No app to install and no custom firmware: the sender copies a few messages into the stock MeshCore app, and the receiver pastes them into a web page to get the picture back.

**Live app:** https://USERNAME.github.io/meshpix/

## How it works

MeshCore only carries short text. Meshpix shrinks a picture to a few hundred bytes, turns the bytes into text, and splits the text into message-sized pieces. A 64-pixel photo usually fits in 2 to 4 messages.

Everything runs in the browser. Pictures never leave your device except as the messages you send yourself.

## Sending

1. Open the app, pick a picture (or take one), and crop it.
2. Choose a preset, or set size and quality yourself. **Fit into a set number of messages** picks the settings for a message budget.
3. You can choose to include the link to the website (https://cbtbeast2000.github.io/MeshPix/#receive) in the "Link to this page" Section to appear in the heads up message
4. Optionally send the heads-up message first so your friend knows what's coming.
5. Press **Copy message 1**, paste it into MeshCore, send. Repeat until every tile is filled.

## Receiving

1. Copy each message starting with `~m` or `~b` from MeshCore.
2. Paste it into the **Receive** tab, one at a time or all at once, in any order.
3. The picture draws from the top as messages arrive. Recovery messages fill in for any that go missing.
4. Save or copy the finished picture.

## Features

- **Two codecs.** *Photo* is a JPEG-style DCT codec with an adaptive arithmetic coder, built for pictures under 1 KB where normal JPEG headers alone would be too big. *Pixel art* uses a 2–64 colour palette with optional dithering, for drawings, maps, logos and screenshots.
- **Loss recovery.** Optional Reed–Solomon recovery messages: with 5 picture messages and 2 recovery messages, any 5 of the 7 rebuild the picture.
- **Live preview** that runs the receiver's decoder on the actual bytes, so what you see is what they get.
- **Crop, rotate, zoom** and brightness, contrast, colour and sharpen adjustments.
- **Airtime estimate** for US, EU/UK or custom LoRa settings, with a suggested wait between messages.
- **Progressive decoding**, remembered across page reloads.
- **Works offline** as a single HTML file.

## Running it yourself

The app is one self-contained HTML file. Host `index.html` anywhere static (GitHub Pages, Netlify, Cloudflare Pages), or open it straight from disk.

There's also a single-file Python server with no dependencies (Python 3.8+):

```
python3 meshpix.py --open              # http://localhost:8000
python3 meshpix.py --host 0.0.0.0      # let phones on your Wi-Fi connect
python3 meshpix.py --export index.html # write out the static page
gunicorn meshpix:app                   # or run it under any WSGI server
```

The "Paste from clipboard" button needs HTTPS or localhost. Over plain http on a LAN, receivers paste with long-press instead.

## Message format

Each message is a 7-character header followed by data:

| Characters | Meaning |
|---|---|
| `~` | Marker |
| `m` / `b` | Alphabet: `m` is a mesh-safe base 85, `b` is Base64 |
| 2 chars | Picture id |
| 1 char | Message index (base 62) |
| 1 char | *k*, number of picture messages |
| 1 char | *n*, total messages including recovery |
| rest | Data, the same length in every message of a picture |

The data carries a small container: version and codec byte, length, caption, image payload and a CRC-16. Recovery messages are a systematic Cauchy Reed–Solomon erasure code over GF(256).

## Etiquette

Every message on a public channel is repeated by each repeater that hears it. Please send pictures as direct messages or on a private channel, keep them to a handful of messages, and leave a gap between sends.

## Troubleshooting

- **A message isn't recognised:** it was probably cut short. Lower *Characters per message* under Message settings.
- **Checksum mismatch:** messages from two pictures got mixed. Remove the picture and paste again.
- **Symbols getting mangled by an app:** switch the alphabet to Base64.

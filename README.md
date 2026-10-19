# Django-Server-IoT

A small Django web server that lets you control a connected IoT device from the browser.

The device blinks on a duty cycle. Instead of reflashing the firmware to change the timing,
you open the page, type the on and off durations in milliseconds, and submit. The server
validates and stores them, and the microcontroller reads them back on its next poll. The
browser and the device therefore share one source of truth: the server.

Built as a university project for an IoT course.

## How it works

| Route | Purpose |
|---|---|
| `/` | HTML form (hand-written HTML + CSS) to set the on/off timing |
| `/getValues/` | Plain-text endpoint the microcontroller polls to read the current values |
| `/admin/` | Django admin, to inspect the stored record |

The values live in a single-row `values` model (`timeOn`, `timeOff`) stored in SQLite
through the Django ORM. Input is checked before it is written, so a non-numeric or empty
submission leaves the previous values in place. On first run the record is seeded with
500 ms / 500 ms.

## Running it

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver 0.0.0.0:8000
```

Open <http://localhost:8000/> to set the timing. The device should poll
`http://<server-ip>:8000/getValues/`, which responds with:

```
On=500&Off=500 end
```

Bind to `0.0.0.0` so the microcontroller on the same network can reach the server, and add
its address to `ALLOWED_HOSTS` in `iotproject/settings.py`.

## Stack

Python · Django · SQLite · HTML5 · CSS3

## Known limitations

- The timing is written on a `GET` request, which is convenient for the device but is not
  how a write should be modelled; a `POST` form and a separate read endpoint would be correct.
- `/getValues/` returns a rendered template rather than JSON. A real REST endpoint would be
  a better contract.
- No automated tests.
- Single hardcoded record (`id=1`), so the server controls exactly one device.

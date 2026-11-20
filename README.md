# Bandcamp retrieval plugin

A Melodex retrieval plugin (contract v1.0) that downloads albums and songs you've bought on
[Bandcamp](https://bandcamp.com), in FLAC or any other format Bandcamp offers for them.

Bandcamp lets you download what you've bought as often as you like, in the format you choose, with nothing locked.
The plugin does what you'd do on your collection page, so it only ever fetches music that's yours. Bandcamp has no API
for this, so it reads the same pages and calls your browser does, and a change on Bandcamp's side can break it until
the plugin is updated.

It runs beside [Melodex](https://github.com/Dextrd/Melodex) as a container of its own, which Melodex reaches over the
[retrieval plugin contract](https://github.com/Dextrd/Melodex/blob/main/docs/contracts/README.md).

## What it does

- **Sign in.** With the `identity` cookie of a browser signed in to Bandcamp. The health check says whose collection
  it sees and how much is in it, hidden purchases included.
- **Search.** For an album: the albums in your collection named like it. One is offered only when it has a track for
  every requested one, matched by title, position and length; Bandcamp doesn't give barcodes or ISRCs. For a song:
  the song if you bought it on its own, or on an album of the artist's you bought.
- **Fetch.** Asks Bandcamp for the purchase's download in the chosen format, waits while Bandcamp prepares an album's
  zip, downloads it, and keeps the requested tracks, named `<disc>-<position> <title>.<extension>`. Covers and
  anything else in the zip are left out; Melodex has its own covers. Each download is kept in the staging volume's
  `.bandcamp` folder for 30 minutes, because Melodex asks for the missing songs of an album you have some of one at
  a time, and each would otherwise download the whole album again.
- **Cancel.** Stops a download between chunks.

## Settings

| Setting | Default | Meaning |
| --- | --- | --- |
| `identity` | | The value of the `identity` cookie for bandcamp.com, from a browser signed in to Bandcamp (in the browser's developer tools, under Storage or Application → Cookies). Signing out of Bandcamp in that browser ends it. Needed. |
| `format` | `flac` | One of Bandcamp's download formats: `flac`, `alac`, `wav` or `aiff-lossless` (lossless), or `mp3-320`, `mp3-v0`, `aac-hi` or `vorbis`. |
| `minScore` | `0.6` | Albums and songs that match less well than this aren't offered. |

## Adding it to Melodex

Add this under `services:` in the compose file you run Melodex with, beside `core` and `beets-worker`, so it shares
their `staging` volume:

```yaml
  retriever-bandcamp:
    image: ghcr.io/dextrd/melodex-retriever-bandcamp:latest
    container_name: melodex-retriever-bandcamp
    user: "${PUID:-1000}:${PGID:-1000}"
    restart: unless-stopped
    volumes:
      - staging:/staging
```

Then start it with `docker compose up -d`. `latest` follows each release; to choose when it changes, use a version
such as `2` or `2.0.0` instead. On SELinux hosts with podman, give it the same `security_opt` as Melodex's own
containers.

Then add `http://retriever-bandcamp:8080` under Settings → Plugins in Melodex (it's one of the templates under Add),
and paste the cookie into its settings.

`BANDCAMP_URL` points the plugin somewhere other than bandcamp.com; Melodex's sandbox uses it for a stand-in.

## Tests

```bash
pip install -r requirements.txt -r requirements-dev.txt
pytest
```

The tests answer for Bandcamp with a fake, so they need no account. They make audio with `ffmpeg`.

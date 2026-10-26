# Media index

Videos and other large media are kept outside Git. This file records what exists, which project it belongs to, and where it is held.

| Title | Project | Type | Description | Location |
|---|---|---|---|---|
| AFTERTONE demo | [AFTERTONE](aftertone/README.md) | Project demonstration | 34-second, 1920×1080 clip of the AFTERTONE interface: keyboard terminal, an invite-code screen, delivered message cards, and a summary of the message path (P-256 ECDH, HKDF-SHA256, AES-256-GCM, 12-byte IV). | Not hosted. Local file `aftertone-demo-github.mp4` (3.8 MB), held by the owner. No public URL yet. |

## Notes on the entry above

- File: `aftertone-demo-github.mp4`, H.264 video with AAC audio, 3,957,498 bytes.
- SHA-256: `fa7fbeff18a9e8500fbdaa168c279ec20051e53fe0be32970a7cb8f8f1e6d883`
- The description is based on four frames sampled at 8-second intervals, not a full viewing.
- Those frames show no fingerprint-verification step, and the invite link shown uses a different hosting hostname from the one in the AFTERTONE README. The video may not match the current interface.

## Adding an entry

1. Add a row with the title, project, type, a one-sentence description, and the location.
2. Use one of these types: development recording, project demonstration, tutorial, walkthrough.
3. Put a real URL in Location only once the video is hosted. Until then, say where the file is held.
4. Record the date or version it shows if that is known.
5. Do not commit the video file.

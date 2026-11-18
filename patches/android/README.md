# Android compatibility patches

This directory contains focused patches applied to the Eden source during the Galaxy M23 Android build.

- `0001-user-prod-keys-reimport.patch`: restores the user-facing production-keys replacement action.
- `0002-m23-saf-content-uri-mmap.patch`: opens Android Storage Access Framework `content://` sources through the Android content-URI file-descriptor helper instead of attempting a filesystem-path mmap.

The Android M23 workflow validates each patch with `git apply --check` before applying it. Gradle dependency resolution is cached between workflow runs, and transient Maven Central HTTP 429 responses are retried with backoff.
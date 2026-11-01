# Buffered Pool iterators: releasing completed registrations

![Victorian engineering illustration: a pool registry, completed work whose entries remain, and the corrected removal of completed entries.](infographic.png)

A buffered `Pool.imap()` or `Pool.imap_unordered()` iterator uses a semaphore
to limit queued work. The pool keeps a registry of these semaphores so it can
wake blocked task submission during shutdown. The ledger and brass tokens in
the illustration represent these Python objects, not operating-system handles.

When all results arrive, the old implementation clears the iterator's reference
to its pool. Later cleanup can no longer remove its semaphore from the pool's
registry. Repeating completed operations therefore leaves unused registrations
in a pool that stays alive.

The patch removes the registry entry when the operation completes, before
clearing the pool reference. Active work and buffered results keep their normal
behavior. Shutdown takes a list snapshot of the registry while releasing entries,
matching the existing approach in `close()`.

See [validation and reproduction](validation.md) for the measured behavior,
regression coverage and runtime limitations. The [image prompt](image_prompt.md)
records the specification used with the native image generation tool.

This illustration is conceptual. It makes no claim about official priority,
production memory usage, native CPython 3.16 validation or a free-threaded build.


# Handling placeholder issues

## Implementing an owner: placeholder issue

Build it for real — actual working code, not a stub — using a sensible, clearly-fake-looking
default in place of the thing only the user can really decide: a reasonable color palette, generic
but coherent copy, a plain placeholder icon/image. The point is the feature is fully runnable and
demoable, just not final.

Note in the issue's Notes what's placeholder and what the user needs to supply to finalize it
(e.g. "using a default blue/gray palette pending your brand colors" or "logo is a generic
placeholder icon — swap in `/assets/logo.svg` once you have the real one"). When you mark it done,
append `⚠️ placeholder` to its `Status` cell in `issues/INDEX.md` (see `doc-to-issues`'s
convention) instead of plain `done`.

## Resolving a placeholder

When the user gives you the real thing a `done ⚠️ placeholder` issue was standing in for (actual
brand colors, real copy, a logo file), swap it in: replace the placeholder implementation, remove
the "placeholder pending..." note from the issue, and clear the `⚠️ placeholder` marker in
`issues/INDEX.md` back to plain `done`. Take it through code review again if the change is
substantive enough to warrant it (a full visual overhaul) — use judgment the same as for any other
change, it doesn't need the full definition-of-done ceremony for a one-line color swap.

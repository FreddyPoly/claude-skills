# Resolving a hard blocker (owner: user)

When the user provides what an `owner: user` issue needs (a decision, a completed account signup,
a credential now in place), implement it like any other issue: acceptance criteria, tests,
code/security review, done.

If what they gave you is non-secret information (a company name, a pricing decision, a domain),
record it in `SPEC.md` the same way the interview would — this is a real decision, not just this
one issue's detail. If it's a credential or secret, never write the actual value into `SPEC.md`,
the issue file, or anywhere else — record only that it's been configured and where (e.g. "Stripe
key is set as `STRIPE_SECRET_KEY` in the environment"), consistent with this pipeline's security
handling elsewhere.

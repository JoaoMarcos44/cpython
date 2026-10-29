# ChainMap reverse union

This illustration accompanies [CPython PR #160932](https://github.com/python/cpython/pull/160932), which addresses [issue #160929](https://github.com/python/cpython/issues/160929).

![Illustrated comparison of ChainMap reverse union. The first Counter layer has no stored entries but returns zero for a missing x. The next layer stores x equal to five. The old reverse union copied five; the fix reads through the ChainMap view and preserves zero.](infographic.png)

## What the picture shows

A ChainMap reads its mappings in order. In this example, an empty Counter already answers zero for the missing key. The deeper mapping stores five, but a normal lookup never needs that value.

The old reverse union merged the mappings' stored entries and returned five. Updating the result from the ChainMap view with `m.update(self)` preserves the zero returned by the lookup. The correction is specifically to reverse union, such as `{} | chain`; it does not change Counter's missing-key behavior.

Normal reads can invoke default factories and insert keys into a defaultdict. Reading through long disjoint chains can also require quadratic lookup work. These tradeoffs are disclosed in the PR.

## Validation

The [native CPython 3.16 validation](https://github.com/JoaoMarcos44/cpython/actions/runs/40960153377) ran ten hash seeds. Each run reported 286 cases, 284 passes, two skips, zero failures, and zero errors. Both new regression tests were explicitly present and passed in every run. These are repeated executions of the same selected suites, not ten distinct environments.

The illustration is an AI-generated explanation, not proof of test results or maintainer approval. The text description and linked run are the accessible, authoritative explanation and evidence. This asset branch is separate from the source patch.

## Contribution guidance

- [CPython contribution workflow](https://devguide.python.org/getting-started/pull-request-lifecycle/).
- [CPython AI-tool policy](https://devguide.python.org/getting-started/ai-tools/).
- [GitHub image formatting](https://docs.github.com/en/get-started/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax#images).

The PR retains its issue-prefixed title, issue link, regression tests, NEWS entry, AI disclosure, and focused three-file source diff. The image is supplemental material in the PR description. The CLA bot reports a pending agreement for the commit author, which the author must personally resolve at [cla.python.org](https://cla.python.org/). The author must also review AI-assisted work in detail, as required by the policy. No merge or human approval is claimed.

## Generation

Generated with the native image-generation tool from a local `pr_infographic_prompt.py` brief. The illustrative theme uses a layered card catalogue, ink outlines, and paper-cut shapes. Final output is `infographic.png`; the original generated file is preserved.

# Skills

Reusable Claude Code skills that pair with the [framework](../framework/) house style.

## `bencium-innovative-ux-designer/` — vendored, third-party

A UX-design skill that enforces a **design-thinking protocol** (ask about purpose /
users / tone / constraints first, then commit to a bold, non-generic aesthetic) plus
high-fidelity visual, motion, accessibility, and responsive guidance.

- **Author:** bencium (bencium.io)
- **Source:** <https://github.com/bencium/bencium-claude-code-design-skill>
- **Listing:** <https://www.ui-skills.com/skills/bencium/bencium-innovative-ux-designer/>
- **Vendored:** copied verbatim (no upstream LICENSE file was present at the time of
  vendoring — credit retained here; replace with the upstream terms if/when published).
  Files: `SKILL.md`, `ACCESSIBILITY.md`, `MOTION-SPEC.md`, `RESPONSIVE-DESIGN.md`,
  `DESIGN-SYSTEM-TEMPLATE.md`.

### How it relates to our house style

The skill is **method**; our [`design-system.md`](../framework/design-system.md) is the
**defaults**. Use the skill's reasoning (interrogate the brief, choose a direction, avoid
AI-slop), then apply our constraints: dark **navy** canvas, **one gold** accent, our
component kit. Where the skill is generic (it's web/frontend-oriented), the framework
docs win for anything Android- or brand-specific.

The skill's `DESIGN-SYSTEM-TEMPLATE.md` is a *template* for deriving a token set — it is
**not** our token source. Our colours live once in `ui/theme/Color.kt`; the docs only
reference tokens by role (see design-system.md).

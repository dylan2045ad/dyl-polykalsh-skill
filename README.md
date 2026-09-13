# Dyl PolyKalsh skill

Grok skill that researches Polymarket and Kalshi markets resolving within 48 hours, picks the higher-probability +EV side, and splits remaining balance evenly across multiple bets capped at $1.50.

Trigger in Grok: `Dyl PolyKalsh skill`

Optional args after the trigger: balance, venue (Polymarket / Kalshi / both), theme.

This skill does not place trades. It outputs a plan plus a copy-paste bot prompt.

Install: copy `SKILL.md` and `references/` into `~/.grok/skills/dyl-polykalsh-skill/`.

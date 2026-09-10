# SkillSwap — full application build

Build the complete SkillSwap app: the accounts and skill-chain logic from your uploaded project, plus every feature from the HTML prototype, all in the original green (emerald/forest) look — no violet or blue.

## What you'll get

A signed-in workspace with a side navigation that switches between:

1. **Dashboard** — stats (matches found, active chains, trade hours, community rank), quick actions, recent activity.
2. **My Skills & Profile** — skills you teach and want to learn, proficiency levels, bio, avatar initials.
3. **Learning Goals** — target skills, priority ranking, target dates, progress.
4. **Direct Matches** — reciprocal 1-to-1 swap partners across real members, with a "Message" action.
5. **Skill Chains** — loop detection across all members (A → B → C → A), circular SVG diagram plus link cards.
6. **Skill Gap Analyzer** — AI compares your current skills to a career/learning target and lists gaps with suggested next steps.
7. **AI Mentor** — chat assistant for roadmaps and partner suggestions, aware of your skills and the community.
8. **Discover** — filterable directory of all members, what they teach and want.
9. **Messages** — direct conversations between trade partners.
10. **Collaborative Projects** — create/join projects, needed skills, member roster.
11. **Reputation** — trust score, badges, and reviews left after trades.

Plus a public landing page and the existing sign-up / sign-in flow with session persistence, so several people on different computers share one live community.

## Design

Green palette carried over unchanged: primary `oklch(0.6 0.13 133)`, soft green accent surfaces, green active nav states, badges and rings. Sora headings + Inter body, light neutral background, soft-shadow cards, pill buttons — the prototype's layout and density, recolored to green. Collapsible sidebar on desktop, slide-over drawer on mobile.

## Technical notes

**Backend (Lovable Cloud):** enable Cloud, then one migration carrying schema, grants, RLS, and triggers.

- `profiles` — id, display_name, bio, headline, avatar_color, trust_score, trade_hours; readable by all signed-in users, writable by owner.
- `skills` — user_id, name, kind (`teach` | `learn`), proficiency, category.
- `learning_goals` — user_id, skill, priority, target_date, status.
- `messages` — sender_id, recipient_id, body, read_at; visible only to the two parties.
- `projects` + `project_members` — title, summary, needed_skills[], owner_id.
- `reviews` — reviewer_id, subject_id, rating, comment; readable by signed-in users, insert restricted to the reviewer.
- `badges` derived in code from trades, reviews, and chain participation.

Every table: `GRANT` to `authenticated` + `service_role`, RLS enabled, owner-scoped write policies, all-signed-in read where the feature is community-wide.

**Skill graph:** keep `src/lib/chains.ts` cycle detection as-is; feed it profiles joined with their `skills` rows instead of the `teaches`/`wants` arrays. Direct matches are the length-2 case.

**AI:** Lovable AI (server-side only) powers the Gap Analyzer (structured output: gaps, priority, suggested path) and the Mentor chat (streaming, message list rendered from parts). Both run through server functions/route so no key reaches the browser.

**Routing:** TanStack file routes under `_authenticated/` — dashboard, skills, goals, matches, chains, gap, mentor, discover, messages, projects, reputation — sharing a sidebar layout. Public `/` landing and `/auth` stay outside the gate. Each page gets its own title/description metadata.

**Data flow:** TanStack Query per view, loaders prefetch, mutations invalidate. Messages poll/refetch on focus.

## Build order

1. Enable Cloud; migration for all tables, policies, triggers.
2. Port green design tokens, app shell, sidebar/header navigation.
3. Profile, skills, learning goals.
4. Discover, matches, chains (SVG loop diagram).
5. Messages, projects, reputation.
6. AI gap analyzer and mentor.
7. Dashboard stats wired to the real tables; landing page polish.

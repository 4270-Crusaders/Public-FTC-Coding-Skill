---
name: ftc-robot-codegen
description: Multi-agent workflow that scaffolds a new FTC (FIRST Tech Challenge) robot codebase. Always clones the official https://github.com/FIRST-Tech-Challenge/FtcRobotController.git as the working codebase first, then clones or reads a team's own past codebase (configured via the TEAM_REFERENCE_REPO_URL placeholder at the top of this file) purely as an architecture reference, then builds out the official base's TeamCode module to match that reference's conventions — no other codebase or template is ever substituted in. Use this whenever the user wants to start a new season's FTC robot code, add or rebuild subsystems/OpModes, or asks to "spin up," "scaffold," or "generate" FTC robot code — even if phrased casually, e.g. "let's build this year's TeamCode," "I need the intake and outtake subsystems," "add a new autonomous OpMode." Before first use, edit TEAM_REFERENCE_REPO_URL to your team's own prior repo. Always clone the official base first, then get the team reference repo, then ask which team number this codebase is for before porting anything from the reference repo, then always run the full PM interview (agents/pm.md) before writing any code, even if the user's initial message already describes the robot — extract what's given, but still confirm every item on the checklist before the backlog is considered final.
---

# FTC Robot Codegen

## Configuration — edit this before use

```
TEAM_REFERENCE_REPO_URL = "<replace-with-your-teams-past-codebase-git-url>"
```

Replace the placeholder above with the git URL of your own team's past FTC codebase (e.g. last season's TeamCode repo) before using this skill. Everything below refers to that value as `TEAM_REFERENCE_REPO_URL`. This is separate from, and never replaces, the official FIRST base repo cloned in step 2 below — that URL is fixed and not meant to be edited.

Three phases, run in order. Do not skip the PM phase, and do not let subsystem heads start writing code before the user has approved the backlog.

## Phase 1 — Official Base Clone, Team Reference Repo, Team Number & PM Interview

This skill always works from two repos, used for two different purposes — never conflate them:

- **The official base** — `https://github.com/FIRST-Tech-Challenge/FtcRobotController.git`, the FIRST SDK template. Its clone becomes the actual working codebase for the rest of this skill; every file Phases 2–3 write lives inside it.
- **The team reference repo** — `TEAM_REFERENCE_REPO_URL`, configured above. This is read-only architecture material: its structure and conventions get carried over into the working clone, but its own clone (if made) is never treated as the project, never merged wholesale into the working clone, and nothing else beyond these two repos is ever substituted, supplemented, or cross-referenced.

1. **Check `TEAM_REFERENCE_REPO_URL`.** If it's still the placeholder (not a real git URL), stop and tell the user to edit the Configuration section at the top of this file with their team's past repo before continuing — don't ask them for a URL in chat as a substitute for editing it. If it's already a real URL, proceed.
2. **Ask where the new codebase should live locally** (a new or empty directory — cloning will create/populate it).
3. **Clone the official base first, before anything else:**
   ```
   git clone https://github.com/FIRST-Tech-Challenge/FtcRobotController.git <target-path>
   ```
   This clone *is* the working codebase going forward. Don't build a separate project from scratch and don't skip this step even if the team reference repo looks like it'd work as a starting point on its own — the official base is always what gets built on top of.
4. **Then get the team reference repo.** Clone it to a separate scratch/reference location outside the working clone (e.g. `<target-path>-reference` or a temp directory) — or, if the user already has a local copy, ask for that path instead of re-cloning:
   ```
   git clone <TEAM_REFERENCE_REPO_URL> <reference-path>
   ```
   This clone is for reading only. Never copy it wholesale over the official clone, and never let its build config or package identity (whatever team number or placeholder it's keyed to) leak into the working clone.
5. **Ask the user two things: what team number this codebase is for, and what team number (or placeholder) is currently baked into the reference repo.** You'll need both for anything you carry over from the reference repo that embeds the old number (see step 7), and for the working clone's own project naming/config. Do not proceed to step 6 without both explicitly — don't assume the old number from the repo name or guess it matches a past season.
6. **Read the team reference repo** to extract its architecture: module layout, subsystem class conventions, OpMode structure (TeleOp vs. Autonomous), hardware map naming conventions, and vendor libraries in use (Road Runner, FTC Dashboard, etc.). The subsystem list and hardware for the new robot will differ from the reference repo's own past robot — it's the *structure*, not the specific subsystems, that should carry over.
7. **Architect the working clone (the official base) to match that structure.** The official base ships nearly empty (sample OpModes only), so this means building out its TeamCode module — package layout, `hardwareMap` access style, state-machine conventions, etc. — to mirror what you read in step 6, with **TeamCode holding all of the new robot's code and controls** (don't split robot logic into a separate module the way some FTC templates do). If you port any file or snippet directly from the reference repo rather than just following its pattern, rebrand any embedded old team number to the new team number from step 5 as part of that transplant, and re-grep the ported content afterward to confirm nothing was missed.

Only once the working clone exists and is architected to match the reference repo's conventions, read `agents/pm.md` and act as the PM described there. Run the interview in full — don't shorten it because the user seems experienced, and don't re-ask the team number or target path since Phase 1 already established them. The user should be grilled every time, on purpose, because incomplete specs are what produce bad subsystem code downstream.

Produce two files in the working project's `.pm/` directory:

- `.pm/backlog.md` — one section per subsystem, structured per the template in `agents/pm.md`
- `.pm/control-mapping.md` — full gamepad1/gamepad2 mapping

(No CAN ID doc for FTC — that's an FRC-only artifact.)

Show both files to the user and get explicit approval before Phase 2. Treat "looks good" as approval; treat any correction as a reason to update the files and re-confirm before moving on.

## Phase 2 — Subsystem Head Code Generation (multi-agent)

Once the backlog is approved, spawn one subsystem-head subagent per subsystem in the backlog — do not predefine the subsystem list yourself; it comes entirely from what Phase 1 produced. Give each subagent `agents/subsystem-head.md`, its one subsystem's section from `backlog.md`, the relevant rows from `control-mapping.md`, and the architecture patterns you gathered from the team reference repo in Phase 1. Subagents write their files directly into the working clone's (official base) TeamCode module, alongside the architecture already built out there — not into a separate scratch location, and not into the reference repo's own clone.

Run these in parallel where your environment supports it. Each subagent writes only its own subsystem class file(s) in TeamCode and a small "integration manifest" — see `agents/subsystem-head.md` for the exact contract. Subagents must never edit shared files (OpMode files, a shared hardware-map constants file) directly, to avoid collisions when running in parallel.

## Phase 3 — Integration

Read `agents/integrator.md` and act as the integrator described there. Collect every subsystem head's manifest, wire each subsystem into the relevant TeleOp/Autonomous OpModes, add the gamepad bindings, and reconcile hardware map names against the robot configuration. Then do a final pass: does every hardware device named in `.pm/backlog.md` actually get initialized from the hardware map? Does every binding in `.pm/control-mapping.md` actually exist in the TeleOp OpMode? Flag and fix any gaps yourself before telling the user it's done.

## Output

Report back with: the team number used, the working project's target path, the team reference repo location, the list of subsystems generated, the OpModes produced, and the location of the two `.pm/` docs (they're meant to outlive this one code-generation pass).

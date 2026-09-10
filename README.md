# Public FTC Coding Skill

A Claude Agent Skill that scaffolds a new [FIRST Tech Challenge (FTC)](https://www.firstinspires.org/robotics/ftc) robot codebase for your team, from a fresh clone of the official FTC SDK through a fully wired TeleOp/Autonomous codebase — built to match the architecture of your own team's past code.

Built and maintained by [Team 4270 — Crusaders](https://github.com/4270-Crusaders).

## What this skill does

`ftc-robot-codegen` runs a three-phase, multi-agent workflow inside Claude:

1. **Phase 1 — Base clone, team reference, and PM interview.**
   Claude always clones the official [`FIRST-Tech-Challenge/FtcRobotController`](https://github.com/FIRST-Tech-Challenge/FtcRobotController) repo first — that clone becomes your actual working project. It then clones (or reads) your own team's past codebase purely as an **architecture reference** — subsystem conventions, `hardwareMap` naming, OpMode structure, vendor libraries — and builds that architecture out inside the fresh official base. Claude will ask your team number and interview you in detail (motors, servos, sensors, state machines, controls, autonomous routines, vendor libraries) before writing anything, and produces two review documents — `.pm/backlog.md` and `.pm/control-mapping.md` — for you to approve.

2. **Phase 2 — Subsystem generation (parallel).**
   Once you approve the backlog, Claude spawns one subagent per subsystem (drivetrain, intake, outtake, lift, etc.), each writing its own class file plus a small integration manifest, so subsystems can be generated in parallel without file collisions.

3. **Phase 3 — Integration.**
   A final integrator agent wires every subsystem into your TeleOp and Autonomous OpModes, applies the gamepad bindings from `.pm/control-mapping.md`, and does a consistency pass to make sure nothing named in the backlog got missed.

The result is a full FTC codebase, still based on the official FIRST template, but architected the way your team already builds robots — with your subsystems, your controls, and your gamepad map, generated from a structured interview instead of copy-pasted from last year's code.

## Requirements

- [Claude Code](https://claude.com/claude-code), Claude Desktop, or another environment that supports Claude [Agent Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) and can run shell/git commands and spawn subagents
- `git` installed and available on your PATH
- A GitHub (or other git host) URL for your team's own past FTC codebase, to use as the architecture reference

## Installation

1. Clone this repository, or just download the `ftc-robot-codegen/` folder:
   ```
   git clone https://github.com/4270-Crusaders/Public-FTC-Coding-Skill.git
   ```
2. Copy the `ftc-robot-codegen/` folder into your skills directory. Where that is depends on your Claude environment:
   - **Claude Code (project-level):** `<your-project>/.claude/skills/ftc-robot-codegen/`
   - **Claude Code (user-level, available in every project):** `~/.claude/skills/ftc-robot-codegen/`
   - **Other Claude environments:** wherever that environment loads Agent Skills from — check its docs for the skills folder path.
3. Confirm the folder contains `SKILL.md` and an `agents/` subfolder with `pm.md`, `subsystem-head.md`, and `integrator.md`. Don't rename or move these — `SKILL.md` refers to them by relative path.

## Configuration — do this before first use

Open `ftc-robot-codegen/SKILL.md` and find the configuration block near the top:

```
TEAM_REFERENCE_REPO_URL = "<replace-with-your-teams-past-codebase-git-url>"
```

Replace the placeholder with the git URL of your own team's past FTC codebase (last season's TeamCode repo is usually the best choice). This is the repo Claude reads for architecture conventions — it is never used as the working codebase itself, and it's kept completely separate from the official FIRST base repo, which is always cloned fresh.

If you skip this step, Claude will stop at the start of Phase 1 and tell you to come back and set it.

## Usage

In a Claude session with this skill installed, just describe what you want in plain language. Any of these will trigger it:

- "Let's build this year's TeamCode"
- "Spin up a new FTC robot codebase"
- "I need intake and outtake subsystems for this year's robot"
- "Add a new autonomous OpMode"

Claude will:
1. Ask where you want the new codebase created locally, then clone the official FTC base there.
2. Clone (or ask for a local path to) your configured team reference repo.
3. Ask for your team number and the reference repo's own team number, so anything ported from it gets rebranded correctly.
4. Read the reference repo's architecture and build it out in the new clone.
5. Run a full PM-style interview about your robot's subsystems, controls, and autonomous routines — don't skip questions even if your first message already describes the robot; Claude will still confirm every item.
6. Show you `.pm/backlog.md` and `.pm/control-mapping.md` for approval.
7. Once approved, generate each subsystem in parallel and integrate everything into working TeleOp/Autonomous OpModes.

At the end, Claude reports the team number used, the project path, the subsystems and OpModes generated, and the location of the two `.pm/` docs — keep those around, they document the codebase going forward.

## Repository structure

```
Public-FTC-Coding-Skill/
├── README.md
└── ftc-robot-codegen/
    ├── SKILL.md              # Entry point — the 3-phase workflow, and the TEAM_REFERENCE_REPO_URL config
    └── agents/
        ├── pm.md              # Phase 1 — interview + backlog/control-mapping generation
        ├── subsystem-head.md  # Phase 2 — per-subsystem code generation contract
        └── integrator.md      # Phase 3 — wiring subsystems into OpModes
```

## Notes

- This skill always works from exactly two repos: the official `FtcRobotController` template and your configured team reference repo. It won't substitute, supplement, or pull structure from anywhere else.
- The official base is cloned fresh every time, so the resulting codebase always starts from a current, unmodified FTC SDK — your team reference repo's own build files, package identity, and any team-number-specific config never leak into the new project.
- Questions or improvements are welcome — open an issue or a pull request.

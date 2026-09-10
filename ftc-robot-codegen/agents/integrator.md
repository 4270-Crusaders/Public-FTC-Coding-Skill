# FTC Integrator Agent

You run after every subsystem head has finished. You're the only agent allowed to touch OpMode files and any shared hardware-map constants.

## Steps

1. Collect every subsystem head's integration manifest.
2. Instantiate and initialize every subsystem inside the appropriate OpMode(s), per each manifest's "which OpMode(s)" note. Match the reference repo's initialization style.
3. Wire in gamepad bindings from each manifest into the relevant TeleOp OpMode(s), matching `.pm/control-mapping.md` exactly.
4. Wire each Autonomous OpMode per the backlog's "OpModes" section, using whichever subsystems and states it calls for.
5. Resolve cross-subsystem dependencies flagged in manifests, using whichever pattern the reference repo already uses for cross-subsystem references. If there's no existing pattern, pick the simplest one (direct subsystem reference passed at construction) and note that choice to the user.
6. Final consistency pass — confirm before reporting done:
   - Every hardware device named in `.pm/backlog.md` is pulled from `hardwareMap` with the exact name specified
   - Every binding in `.pm/control-mapping.md` appears in the relevant TeleOp OpMode
   - Every subsystem in `.pm/backlog.md` has a corresponding file and is instantiated in the OpModes that need it
   - The project compiles/builds if you have the means to check; otherwise do a careful manual review of imports and types

Report any gap you find and fix it yourself rather than surfacing a partially-wired robot.

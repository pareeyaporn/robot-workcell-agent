# TASKS: Talk to a Robot (robot-workcell-agent)

**Timeline:** 4 weeks at about 1–2 hours a day. Each week ends with a working demo.
**Rule:** every checkbox has a "done when" test. A box is ticked only when that test passes.

## 🎯 Definition of done (v1)
- [ ] A spoken instruction runs a full job end to end: parts on the belt get inspected and sorted
- [ ] Chaos mode: the robot recovers from injected failures, with the recovery rate measured
- [ ] Safety shield refuses all unsafe tasks in the benchmark
- [ ] 100-task benchmark run; results and the model comparison published in the README
- [ ] Mission-control dashboard works live
- [ ] Demo GIF (~15 s) in the README, 60–90 s video for LinkedIn, project card on the GitHub profile, LinkedIn Featured, and a CV bullet with real numbers

---

## Week 1: 🏭 Build the factory cell (simulation and motion)

### 1.1 Project setup
- [ ] Package layout: `workcell/{scene,kinematics,skills,perception,agent,safety,chaos,bench,voice,log}.py`, `tests/`, `docs/`
- [ ] Pin dependencies in `requirements.txt`; `MUJOCO_GL=egl` documented
- [ ] GitHub Actions: run `pytest` on every push
  - *Done when:* CI is green on a fresh push

### 1.2 The scene
- [ ] Load the Franka Panda from MuJoCo Menagerie (keep its Apache-2.0 license file) on a table
- [ ] Suction gripper: a small cup at the flange plus a weld constraint switched on and off; attach only if the cup touches a part
- [ ] Three bins with colored labels: A (blue), B (green), Reject (red)
- [ ] Part generator: colors {red, blue, green} × shapes {cube, cylinder}; defects {none, scratch, chip, crack} as texture or geometry marks; ~25% defective
- [ ] Conveyor belt: a strip whose parts move at a set speed (start: kinematic motion; parts spawn at one end on a timer)
- [ ] Cameras: overhead (perception), wrist (inspection), front (cinematic, for demos)
- [ ] Lighting and materials for good-looking renders (soft shadows, matte table, metallic arm)
- [ ] `Workcell.reset(seed, n_parts, belt=True/False)` is reproducible
  - *Done when:* the same seed renders identical images twice

### 1.3 Motion
- [ ] Forward kinematics helpers: gripper pose from joint angles
- [ ] Inverse kinematics: damped least squares to an (x, y, z, yaw) target, with joint limits
- [ ] Trajectories: smooth joint interpolation (minimum-jerk), capped speed
- [ ] Moving-part interception: predict where a belt part will be at grasp time and meet it there
  - *Done when:* IK reaches 95%+ of random reachable targets within 5 mm (test over 200 targets)

### 1.4 First skills
- [ ] `home()`, `pick(x, y)`, `place(bin)`, each returning `{ok, error, details}`
- [ ] Verification from the simulation: part attached after pick; part inside bin after place
  - *Done when:* pick-and-place succeeds on 50 random seeds (static parts) at 95%+, and on belt parts at 80%+

**🎬 Week 1 demo:** a scripted run picks every part (including from the moving belt) and drops it into bin A, rendered from the front camera as a video.

---

## Week 2: 👀 Give it eyes and a brain (perception and agent)

### 2.1 Perception, stage 1 (fast classical vision)
- [ ] Overhead image → HSV color masks → contours → part centers, color, shape (circularity test)
- [ ] Pixel → table coordinates using the camera intrinsics and pose
- [ ] Track parts on the belt across frames (nearest-neighbor tracker with ids)
- [ ] `look()` returns `[{id, color, shape, x, y, on_belt, velocity}]`
  - *Done when:* position error under 1 cm and color/shape accuracy 98%+ over 50 seeds (true simulator positions used only inside the test)

### 2.2 Perception, stage 2 (AI vision inspection)
- [ ] `inspect(part_id)`: move the wrist camera over the part and capture a close-up
- [ ] Send the close-up to a vision-language model with a strict JSON schema: `{defect: bool, type, confidence, reason}`
- [ ] Offline fallback: a simple image check (edge density / dark-pixel ratio) so it runs without a key
  - *Done when:* defect precision and recall measured on 200 labeled close-ups (both methods)

### 2.3 Planner agent
- [ ] Tool definitions (JSON schemas) for `look`, `inspect`, `pick`, `place`, `home`, `report`
- [ ] Agent loop: instruction → model chooses tools → results returned → repeat until `report`, with a step limit (e.g. 60)
- [ ] Job memory: a structured list of parts and their status (todo, done, failed, skipped) passed back to the model each turn
- [ ] System prompt: use only the tools; check results; re-look after any failure; state assumptions for ambiguous requests; refuse unsafe ones
- [ ] Offline planner: rule-based parser for common instructions using the same tools
- [ ] Event log: every step written as JSON lines (time, tool, args, result, image refs) for the dashboard and benchmark
  - *Done when:* 10 hand-written instructions across tiers 1–3 complete correctly with the LLM planner

**🎬 Week 2 demo:** `python -m workcell.demo "good blue parts to B, damaged to reject"` with the overhead view, inspection close-ups, and the step log.

---

## Week 3: 🛡️ Make it safe, robust, and talkative (safety, chaos, voice)

### 3.1 Safety shield
- [ ] Check every tool call before execution:
  - [ ] Workspace bounds and minimum table clearance
  - [ ] Joint speed and acceleration limits
  - [ ] Predicted self-collision and collision with bins (check the planned trajectory in a copy of the simulation)
  - [ ] Forbidden zone (an "operator area" marked on the table)
  - [ ] Action budget per job
- [ ] On rejection, return a clear reason to the agent so it can re-plan or refuse
- [ ] Request-level refusals for tier-6 tasks, with an explanation
  - *Done when:* 0 safety violations across the benchmark and 100% of tier-6 tasks refused

### 3.2 Chaos mode
- [ ] Injectors (each with a probability setting): pick slip mid-move, part knocked off the table, blocked bin (lid closes), camera noise or blur, belt speed change
- [ ] The agent must notice (through verification and re-looking) and recover
- [ ] Log each injected fault and whether and how it was recovered
  - *Done when:* recovery rate measured at chaos levels 0%, 10%, and 25%

### 3.3 Voice
- [ ] Push-to-talk audio → faster-whisper (local) → instruction text (reuse the voice-agent pipeline)
- [ ] Spoken summary at the end of a job (text-to-speech), with an option to turn it off
  - *Done when:* 10 spoken test instructions transcribed and executed correctly

**🎬 Week 3 demo:** speak "sort everything on the belt"; chaos knocks a part off; the robot notices, finds it, and finishes; the shield blocks one unsafe move on screen.

---

## Week 4: 📊 Prove it and show it (benchmark, dashboard, launch)

### 4.1 Benchmark
- [ ] 100 tasks across the six tiers (see README), each with an automatic checker comparing final bin contents with the expected result
- [ ] Runner: `python -m workcell.bench --planner {offline,llm} --model <name> --chaos <level> --seeds ...`
- [ ] Metrics: success rate, sorting accuracy, defect precision/recall, recovery rate, refusals, steps, simulated time, wall-clock latency, API cost
- [ ] Model comparison: run 2–3 AI models (a small, medium, and large model) on the same seeds
- [ ] Charts: success by tier, success vs. chaos level, cost vs. success per model
  - *Done when:* results tables and charts committed under `docs/results/`, and the README filled in

### 4.2 Mission control (Streamlit)
- [ ] Input: microphone button and text box, plus example tasks
- [ ] Live panels: overhead camera with detection boxes, wrist close-ups with inspection verdicts, front cinematic view
- [ ] Plan timeline and step log (✓ success, ✗ failure, 🔁 recovered, 🛡️ blocked by the shield)
- [ ] KPI tiles: parts sorted, accuracy, throughput, recoveries, safety blocks
- [ ] Replay mode: load any event log and replay a run
  - *Done when:* a full job runs live in the dashboard without errors

### 4.3 Launch
- [ ] Record the demo: 15-second GIF (README) and 60–90-second video (LinkedIn), with the voice command, belt, inspection, a recovery, and a shield block
- [ ] README: results, charts, architecture, how to run, limitations (it's a simulation; the suction gripper simplifies grasping)
- [ ] Code quality: type hints, docstrings, tests passing, CI green
- [ ] GitHub profile: third project card with the GIF
- [ ] LinkedIn: Featured video plus a short post about what you learned
- [ ] CV: add a bullet with the real numbers to the job-search profile

---

## 🌟 Stretch goals
- [ ] Real hardware: SO-101 arm with LeRobot running the same skills interface (sim-to-real)
- [ ] Two arms with a handoff between them
- [ ] Learning-based grasping (train a small policy in simulation)
- [ ] Natural-language "why" questions about any past job ("why did you reject part 7?"), answered from the event log
- [ ] Web 3D viewer of the workcell

## 📝 Notes
- Honest numbers only: every result comes from a script run with recorded seeds.
- The agent never sees simulator ground truth; only tests do.
- Never commit API keys (`.env` is gitignored). Everything must run with the offline planner and no key.
- Commit small and often; each week's demo must run from a fresh clone.

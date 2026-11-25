# TASKS: Talk to a Robot (robot-workcell-agent)

Goal: a working, demo-ready project in **3 weeks**, at about 1–2 hours a day. Every week ends with something that runs and can be shown.

**Definition of done for the project**
- [ ] One command runs a full task from an English instruction to a sorted table
- [ ] Dashboard shows the plan, the camera view, and the step log live
- [ ] Evaluation results on 50 tasks published in the README
- [ ] Demo GIF (about 15 seconds) at the top of the README, plus an MP4 for LinkedIn
- [ ] Added to the GitHub profile README and LinkedIn Featured

---

## Week 1: The robot moves (simulation and skills)

**Setup**
- [ ] Create the repo `robot-workcell-agent`, a Python package `workcell/`, `requirements.txt`, `.gitignore`, `.env.example`
- [ ] Install MuJoCo and confirm offscreen rendering works (`MUJOCO_GL=egl`)

**The workcell scene**
- [ ] Load a 7-joint arm (MuJoCo Menagerie Franka Panda, Apache-2.0) on a table
- [ ] Add a suction gripper at the end of the arm (a weld constraint that switches on and off)
- [ ] Add 3 bins (A, B, Reject) and 6–10 parts: random colors (red, blue, green), random positions, some marked "damaged" (a visible notch or dark stripe)
- [ ] Add an overhead camera and a front "beauty" camera for demo videos
- [ ] `scene.reset(seed)` gives a reproducible random layout

**Motion and skills**
- [ ] Inverse kinematics: move the gripper to an (x, y, z) target with damped least squares
- [ ] Smooth motion: interpolate joint positions and avoid jerky moves
- [ ] Skills, each returning `{ok, message}`:
  - [ ] `home()`
  - [ ] `pick(x, y)`: approach from above, descend, suction on, lift, and check the part is attached
  - [ ] `place(bin)`: move over the bin, lower, suction off, and check the part landed in the bin
- [ ] Workspace limits: refuse targets outside the table or below its surface
- [ ] Tests: `pick` then `place` succeeds on 20 random seeds

**Week 1 demo:** a script picks every part and drops it in bin A, saved as a short video.

---

## Week 2: The robot sees and thinks (perception and the agent)

**Perception**
- [ ] Render the overhead camera image and find parts by color (HSV thresholds and contours with OpenCV)
- [ ] Convert pixel positions to table (x, y) using the camera's known pose
- [ ] Detect "damaged" parts from the visual marker
- [ ] `look()` skill returns a list of parts: `{id, color, damaged, x, y}`
- [ ] Test: perception positions within 1 cm of the true positions on 20 seeds (true positions used for testing only)

**Agent (planner)**
- [ ] Define the tools for the LLM: `look`, `pick(part_id)`, `place(bin)`, `home`, `finish(summary)`
- [ ] Agent loop: instruction → tool calls → results back to the model → repeat until `finish`, with a step limit (for example 40)
- [ ] System prompt with safety rules: only use the tools, refuse impossible or unsafe requests, look again after any failure
- [ ] Offline rule-based planner with the same tools (no API key needed; used for tests and as a fallback)
- [ ] Log every step (tool, inputs, result, time) to JSON for the dashboard and evaluations
- [ ] Recovery: randomly drop 10% of picks (a "slip") and confirm the agent notices and retries

**Week 2 demo:** `python -m workcell.demo "put the blue parts in bin B"` works end to end.

---

## Week 3: Prove it and show it (evaluations, dashboard, polish)

**Evaluation**
- [ ] 50-task evaluation set: varied instructions (by color, damaged vs. good, counts like "two red parts", impossible requests that should be refused)
- [ ] Automatic checker: compare final bin contents with the expected result
- [ ] Report: success rate, average steps, recoveries, refusals handled correctly, average time
- [ ] Compare the LLM planner with the offline planner and write down what the LLM handles better (ambiguous or complex instructions)

**Dashboard (Streamlit)**
- [ ] Text box for the instruction, plus example buttons
- [ ] Live camera view (overhead and front), the plan, and the step log with ✓ and ✗ marks
- [ ] Final summary card: what moved where, steps, and time

**Polish**
- [ ] README: fill in results, architecture, and how to run
- [ ] Record the demo: about 15-second GIF for the README and a 30–60-second MP4 for LinkedIn
- [ ] Clean code, docstrings, `pytest` passing, and a GitHub Actions workflow that runs tests
- [ ] Add to the GitHub profile README as the third project card (with GIF) and to LinkedIn Featured
- [ ] Add a CV bullet to the profile (via the job-search repo) with real results

---

## Stretch goals (after the 3 weeks)
- [ ] Vision-language model for perception ("find anything that looks scratched")
- [ ] Voice commands (reuse the Whisper pipeline from the voice agent project)
- [ ] Two-arm handoff, or a conveyor belt with moving parts
- [ ] Real hardware: SO-101 arm with LeRobot, running the same skills interface

## Notes
- Keep the agent honest: it only acts through skills, and skills report real success or failure from the simulation.
- Never commit API keys. Use `.env` (gitignored).
- Commit small and often; each week's demo should run from a fresh clone.

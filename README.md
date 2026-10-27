# LabSync

**Research operations for shared lab resources: one request from a researcher, one coordinated plan, and a clearer view of how a department uses its equipment.**

LabSync is a working prototype built for the C3 AI University Accelerator hackathon. It explores how AI agents can take on the coordination work of running an experiment, while people keep control of every decision.

## The problem

Running a single experiment in a life-sciences lab often means coordinating several separate systems. A researcher who needs three hours on a confocal microscope also needs a prep room beforehand and enough reagents on hand. The microscope may already be booked by another lab, and the reagents may take days to arrive. Each of these lives in a different calendar or ordering system, and the researcher is the one who has to tie them together.

Departments face the same problem at a larger scale. One microscope can be fully booked while an identical one down the hall sits mostly idle. Two labs can order the same antibody separately in the same week and each pay full price and shipping. These patterns are hard to see when every lab's bookings and orders stay separate.

## What LabSync does

LabSync offers two views of the same data.

**For a researcher**, LabSync turns a plain-language request into a plan. For example:

> "I need to image GFP-tagged cells Tuesday morning: three hours on a confocal, a prep room beforehand, and enough anti-GFP antibody and coverslips. I need the results by Wednesday."

From that request it:

- works out the steps (sample prep, then imaging) and the supplies they need;
- checks which instruments can do the job and which ones the researcher is trained to use;
- finds times that respect cleaning gaps between sessions and staffed hours;
- checks stock and delivery lead times for the supplies.

It then offers a ranked set of options. If the best time overlaps another lab's session, and that session was marked flexible, LabSync can ask that lab's manager whether they will move it. Nothing moves until they agree.

**For a department administrator**, LabSync looks across all labs and reports a small number of findings. Each finding comes with its arithmetic, and each needs approval before anything changes. The current findings are:

- **Shared purchasing.** Two labs are ordering the same item. Combining the orders reaches a volume price and pays one shipping charge, for an estimated $95 saving in the demo data.
- **Overloaded equipment.** One confocal microscope is 92.5% booked while a second is at 20%. The finding shows how much flexible time could move between them.
- **Underused equipment.** A flow cytometer is used for about 10% of its staffed hours. The finding suggests opening it to other labs rather than drawing conclusions from one quiet week.

It also forecasts next week's demand for the busiest instruments from the previous six weeks.

## Design principles

- **People decide; AI advises.** Three AI agents interpret requests, advise a lab manager on a proposed move, and write up department findings. They can only suggest. Every booking, move and order request needs a person's approval.
- **Rules live in code, not in the model.** Availability, permissions, training requirements, cleaning buffers, prices and savings are calculated by ordinary, tested code. The model never decides whether a slot is free or how much money is saved.
- **Everything is traceable.** Each change records who made it, when, and what it changed. The history is visible in the app.
- **Privacy between labs.** A researcher sees other labs' bookings only as "Booked", without names or details.
- **Nothing leaves the system.** LabSync never places real orders, sends email or charges a grant. A purchase is only ever a request.

## Try it

You need Python 3.9 or newer. There are no packages to install.

```sh
git clone https://github.com/aaryandevgun/LabSync.git
cd LabSync
python3 -m labsync_app.web --fresh --empty-calendar
```

Then open http://127.0.0.1:8000.

1. **A simple booking.** The page opens as Morgan, a researcher. Click **Book the confocal**, press **Generate plan**, then press **Book this time**.
2. **A booking that needs another lab.** Restart without `--empty-calendar` to load a busy week. As Morgan, choose **Plan an imaging experiment**. The best time overlaps Chen's session, so hold it and ask Chen to move. Use **Signed in as** in the sidebar to switch to Chen and approve the request, then switch back to Morgan to confirm the booking.
3. **The department view.** Switch to Department admin, open **Insights** and press **Run department review**.

A click-by-click walkthrough is in [docs/DEMO-SCRIPT.md](docs/DEMO-SCRIPT.md).

By default the agents run on an offline, rule-based stand-in, so the demo works without an API key. It understands the example requests and simple ones such as "plate reader for 45 minutes Monday at 10". To run the agents on Claude, set `ANTHROPIC_API_KEY` and start the app with `--live`.

## About the data

All data in LabSync is synthetic. It describes a fictional life-sciences department with four labs, ten instruments, four rooms and a small supply catalog. The people, bookings, prices and forecasts are invented for the demo. LabSync is not connected to any university system, and the savings it reports are estimates from the demo data, not measured results.

## How it's built

LabSync is a standalone Python application that uses only the standard library. The web interface is plain HTML, CSS and JavaScript. Data is stored in a local JSON file.

| Folder | Contents |
|---|---|
| `labsync_core/` | Scheduling, eligibility, conflict detection, pricing, savings and forecasting |
| `labsync_tools/` | The actions agents and people can take, with permission checks and the audit log |
| `labsync_app/` | The coordinator that runs the agents, and the web app |
| `agents/` | Agent instructions and the structured formats their answers must follow |
| `data_gen/` | Generator for the synthetic demo data |
| `tests/`, `evals/` | Unit, integration and browser tests, plus agent evaluation suites |
| `docs/` | Product specification, design notes and the demo script |

Run the tests with:

```sh
python3 -m pytest tests -q
```

The browser test runs when Node and Playwright are installed, and skips itself otherwise.

## Limitations and next steps

- LabSync runs on synthetic data only. Using it for real would need access to a department's booking and purchasing systems, and their approval.
- The offline agents understand a limited range of requests. Free-form requests need the `--live` mode with Claude.
- The forecast is a simple weighted baseline, meant to show where forecasting fits rather than to be accurate.
- The hackathon targeted the C3 AI platform. This version runs on its own, and the package in `c3/` is only a placeholder for importing it.

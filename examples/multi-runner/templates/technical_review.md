1. What You’re Given (Assumptions)

Assume the following:
The JIT runner is started by a startup script, e.g. startup.sh.

That script:
Installs the runner

Registers it
Starts it with ./user-data-v2.sh

You can:

Edit user-data-v2.sh

Add new files/scripts under /tmp/runner.
You have access to GitHub runner env vars at job runtime (e.g. RUNNER_NAME, GITHUB_RUN_ID, etc.).

If you need to make reasonable assumptions, document them.

1. Task 1 – Pre-Job & Post-Job Scripts
2.1 Goal

Create a mechanism so that for every job that runs on this runner:
A pre-job script runs before the job.
A post-job script runs after the job, even if the job fails.
We want to act as “a normal user” who just expects this behavior to exist.

2.2 Requirements

Pre-Job Script
Script filename example: pre-job.sh
Runs before the actual job steps.

Should at minimum:
Log that the job is starting

Create a per-run working directory, e.g.
/tmp/runner/workspace/$GITHUB_RUN_ID

Optionally export environment variables for the job

Example expected behavior:
“Before every job starts, create a clean workspace directory for that run.”

Post-Job Script
Script filename example: post-job.sh
Runs after the job completes (success or failure).

Should at minimum:
Log job completion (success/failure if available)
Remove the per-run workspace directory
Write a simple summary file into e.g. /tmp/runner/history/summary.log

Example expected behavior:
“After every job finishes, clean up temp data and write a small summary log.”

2.3 Implementation Constraints / Expectations

You should:
Use the JIT startup script (user-data-v2.sh) to:
Place the pre-job.sh and post-job.sh scripts onto the machine.
Wire them into the runner lifecycle.
Choose a strategy to tie scripts to job start/end, for example (you pick, but explain):
Wrapping the runner process (runsvc.sh or run.sh) with your own script.

Using a job wrapper script that GitHub calls instead of the default runner command.
Any other mechanism that reliably gives you “before job” and “after job” hooks.

Document:

Where scripts live (e.g. /tmp/runner/hooks/pre-job.sh).
How they are called.

How a user could edit them to add custom logic.

1. Task 2 – Collect Runtime Machine Metrics
3.1 Goal

While a job is running, we want to collect machine performance metrics:
CPU usage
Memory usage
Disk usage
We need a simple, inspectable log per job run.

3.2 Metrics Requirements
At a minimum, capture:

CPU
Overall CPU usage percentage
(Bonus) load averages (1m, 5m, 15m)

Memory
Total
Used / free
Disk
Disk usage (used / free) on the main filesystem
(Bonus) disk I/O stats if easy on the OS you assume

3.3 Sampling Requirements

Metrics should be collected only while the job is running.
Suggested interval: every 10 seconds (can be a parameter).
Metrics should stop when the job finishes.

3.4 Output Requirements

Use a per-run directory such as:
/tmp/runner/$GITHUB_RUN_ID/
Log file name example: metrics.log
Each line/entry should contain at least:
Timestamp
CPU usage
Memory used
Disk usage

Format can be:
Plain text (e.g. key=value pairs), or
JSON per line
Make it easy to parse later.

3.5 Integration With Hooks
You are free to choose where to start/stop metrics collection, for example:
Start metrics collection from the pre-job script.
Stop metrics collection from the post-job script.

You might:

Start a background process (e.g. metrics-collector.sh) in pre-job.
Kill or signal it in post-job.
Please document how you do this.

4. What We Want to See From You
4.1 Deliverables

Modified startup script
e.g. startup.sh (or equivalent)

Shows how:
The runner is installed/started
Your hooks and metrics tooling are installed/configured
Hook Scripts
pre-job.sh
post-job.sh

Metrics Collector Script
e.g. metrics-collector.sh
Responsible for looping and writing metrics until told to stop.

Short Documentation (README or section in the issue)
Explain in clear, concise terms:
How the startup script works.
How pre-job and post-job are triggered.
How metrics collection is started/stopped.
Where the metrics will be delivered.

Where a user finds:

Workspace directories
Metrics logs
Summary files
How a user can customize the hooks.

4.2 Things We’ll Evaluate

Correctness
Do pre-job and post-job run at the right times?
Does metrics collection start and stop with the job?

Robustness

Does post-job run even when the job fails?
Are errors handled reasonably (e.g. no catastrophic crashes if a directory already exists)?

Clarity

Is your code readable?
Are comments and docs clear and to the point?

Design Choices

Is your hook mechanism sensible and explainable?
Is the metrics design simple enough to maintain?

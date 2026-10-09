## 1.Continuous integration
In this lesson you will learn what continuous integration (CI) is and what a CI check does on every push.

Back to the pixelpost team. Their worst Fridays happened because broken code sat unnoticed in the shared codebase for days. By the time someone found it, three other changes were stacked on top.

Continuous integration attacks that delay. The idea: every time anyone pushes code, a machine immediately builds the project and runs the tests. Not on Friday. Now.

#### What a CI run looks like
A typical CI run does three things:

- Download the exact code that was just pushed.
- Install the project's dependencies from scratch, so the run does not depend on whatever is - installed on someone's laptop.
- Run the checks: tests, a linter, maybe a build.

If any step fails, the push is marked with a red cross, and the team knows within minutes. The person who pushed still remembers what they changed, so the fix is quick.

#### Why "integration"
The name comes from integrating, meaning combining, everyone's changes into one shared codebase. Doing it continuously means every push, not once a week. Small merges with instant feedback beat one giant risky merge.

### Continuous delivery and deployment
In this lesson you will learn what the CD part of CI/CD means, and the difference between continuous delivery and continuous deployment.

CI answers one question: is this change safe? CD answers the next one: how does a safe change reach users?

Both CD terms describe automating the path from "tests passed" to "running on the server". They differ in one detail: who presses the final button.

#### Continuous delivery: a human presses the button
With continuous delivery, every change that passes CI is automatically packaged and made ready to release. The actual release to users still needs a human decision. Someone clicks "deploy" when the team is ready.

Banks and hospitals often work this way. The automation does all the mechanical work; a person picks the moment.

#### Continuous deployment: no button at all
With continuous deployment, there is no button. A change that passes every check goes to users automatically, minutes after the push. Many web companies release dozens of times a day like this.

The trade is simple. Continuous deployment gives speed and demands very strong tests. Continuous delivery gives a human checkpoint and costs a little speed.

In this skill you will build both kinds of pipeline with GitHub Actions. In the next lesson: a scenario. Your team debates whether automation is worth the effort, and you weigh in.

## 2.What is github actions.
The five building blocks
In this lesson you will learn the five pieces every GitHub Actions setup is made of: event, workflow, job, step, and runner.

In the last topic, the pixelpost team decided to automate their tests. GitHub Actions is the tool they will use. Before writing anything, you need the vocabulary, because every workflow file you ever read uses these five words.

- Event: something that happens in the repository. A push, a pull request being opened, a timer firing. Events are the "when".
- Workflow: a YAML file that says "when this event happens, do these things". One repository can have many workflows.
- Job: a group of steps inside a workflow that runs on one machine. A workflow can have several jobs, and by default they run at the same time.
- Step: one single action inside a job. Run the tests. Install Python. Download the code. Steps run one after another, in order.
- Runner: the machine that actually executes a job. GitHub-hosted runners use a fresh virtual machine for each job. Usage limits and billing depend on the repository and plan.

#### Where each piece lives
In this lesson you will learn which parts of GitHub Actions live inside your repository and which parts GitHub runs for you.

This split matters because it tells you what you control with a git commit and what GitHub handles automatically.

#### Your side: the repository
Workflow definitions live in your repository. They sit in a fixed folder, .github/workflows/, at the root. Repositories can also contain local actions, action metadata, scripts, Dockerfiles, and project files used by those workflows. All are normal files, committed and versioned like other code.

That is the whole point. When the pixelpost team writes their release steps into a workflow file, the process stops living in one person's memory. It is in the repository, reviewed like code, and every teammate gets the same steps.

#### GitHub's side: everything else
GitHub watches your repository for events. When an event matches a workflow, GitHub reads the file, creates the jobs, finds an eligible runner, and starts the job. A GitHub-hosted runner is a fresh virtual machine GitHub prepares for one job and discards afterwards. Self-hosted runners are installed and maintained by their owners and can persist between jobs.

In this course, GitHub-hosted runners remove the need to maintain a build server. Your workflow can still install its dependencies, and self-hosted runners require their own setup and cleanup. GitHub reports the result on the repository page as a green tick or a red cross.

#### How Actions runs a CI/CD pipeline
In this lesson you will learn how the five building blocks turn into the CI/CD pipeline you saw in topic 01.

Remember the CI loop: someone pushes, a machine gets the code, installs dependencies, runs the checks, and reports green or red within minutes. GitHub Actions is one tool that runs that loop. Here is the same loop, labelled with Actions vocabulary:

Every box from topic 01 has an Actions name now. The trigger is an event. The written-down steps are a workflow. The machine is a runner.

#### And the CD part?
`CD fits the same shape. A deploy is just another job with different steps: instead of "run tests", the steps are "package the app" and "copy it to the server".` Continuous delivery and continuous deployment differ only in the event and in whether a human approves the job. Actions supports both, and you will build both later in this skill.

#### Actions is not the only tool
Jenkins, GitLab CI, and CircleCI do the same work. GitHub Actions stands out because it is built into GitHub: no extra server, no separate account, and workflow files live next to the code they test. Since most teams already keep code on GitHub, it is often the shortest path to a working pipeline.

## 3.The .github/workflows folder
In this lesson you will learn where workflow files live and what the file names look like.

GitHub does not search your whole repository for workflows. It looks in exactly one place: the folder .github/workflows/ at the root of the repository. A workflow file anywhere else is ignored, silently, with no error. That silence is a classic beginner trap: the file looks perfect, but nothing ever runs because it sits in the wrong folder.

The rules are short:
- The folder is .github/workflows/, starting with a dot, at the repository root.
- Every file inside ending in .yml or .yaml is treated as a workflow.
- The file name itself is your choice. ci.yml, deploy.yml, nightly.yml - pick names that say what the workflow does.

One repository, many workflows. The pixelpost team might keep ci.yml for tests on every push and deploy.yml for releases. Each file is independent: its own trigger, its own jobs.

In this topic you will practice locally, in folders on this VM, without pushing anything to GitHub. The file format is identical either way, so everything you write here works unchanged once you connect a real repository later.

#### name, on, and jobs
In this lesson you will learn three basic top-level keys used in a workflow file and enough YAML to write one.

Workflow files are YAML. YAML is a plain text format built on two ideas: key: value pairs, and indentation to show what belongs inside what. Indentation uses spaces, never tabs, and must stay consistent. This course uses two spaces per level.
```
name: pixelpost CI
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Say hello
        run: echo "hello from CI"
```

Read it top to bottom:

- name - a display name. It appears in GitHub's user interface next to each run. Optional, but always set it.
- on - the event that triggers the workflow. Here, any push. This is the "when".
- jobs - a map of jobs. Each key under jobs is a job id you invent (test here). A normal job defines runs-on, the runner label, and steps, a list of steps. A job that calls a reusable workflow uses job-level uses instead.

Steps are a YAML list: each step starts with a -. A step that runs a shell command uses the run key, and name gives the step a label in the logs.

Many workflows build on this skeleton with more events, jobs, and steps. name is optional, and other top-level keys can add permissions, defaults, concurrency, environment variables, or a dynamic run name.

### Checking workflows with actionlint
In this lesson you will learn what actionlint is and how to run it on a workflow file.

A broken workflow file fails in an annoying way: you push it, wait for GitHub to try it, and read an error message on the website. Fixing a typo can take several push-and-wait loops. Teams avoid that loop with a linter: a program that reads your file and reports mistakes before you push.

For GitHub Actions, the standard linter is actionlint. It ships as a single binary. It knows the workflow format deeply, so it catches much more than broken YAML:

- misspelled keys, like run-on instead of runs-on
- runner labels that do not exist
- a jobs section with no jobs in it
- shell mistakes inside your run commands

Run it by giving it the path to a workflow file. Create this small example first so the path exists:
```
mkdir -p /root/demo/.github/workflows
cat > /root/demo/.github/workflows/ci.yml <<'EOF'
name: demo CI
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Show message
        run: echo "CI is ready"
EOF
```
`actionlint /root/demo/.github/workflows/ci.yml`

If the file is clean, actionlint prints nothing and exits quietly. That silence means success. If something is wrong, it prints the file name, the line number, and a message:

`ci.yml:5:5: "run-on" section is not allowed. did you mean "runs-on"?`

## 4.push and pull_request events
In this lesson you will learn the two events that trigger most CI workflows: push and pull_request.

The on key you wrote in the last topic held one bare word. It can also hold a list, or a map with settings per event. All three forms are common:
```
on: push
```
```
on: [push, pull_request]
```
```
on:
  push:
  pull_request:
```

The list form triggers on either event. The map form does the same, and leaves room for filters under each event - that is the next lesson.

The list form triggers on either event. The map form does the same, and leaves room for filters under each event - that is the next lesson.

#### When each event fires
- push fires every time commits reach a branch on GitHub. Not when you commit locally, only when the push arrives at GitHub.
- pull_request fires when a pull request is opened, and again on every new push to the branch of that pull request. A push to a same-repository pull-request branch can therefore emit both push and pull_request events.

#### Why teams use both
For pixelpost, push guards the main branch: every change that lands is tested. pull_request tests a change before it lands, so the reviewer sees a green tick next to the code they are reviewing. Most CI workflows you meet in real repositories listen to both.

### Branch and path filters
Without filters, on: push fires for every push to every branch. On a busy repository that wastes runner minutes and buries the results that matter. Filters narrow the trigger.

#### branches: react to some branches only
Filters live under the event in the map form of on:
```
on:
  push:
    branches: [main]
```

Now the workflow runs only when commits reach main. Pushes to any other branch are ignored. The list can hold several branches, and patterns work too: releases/* matches releases/v1, releases/v2, and so on.

#### paths: react when certain files change
```
on:
  push:
    paths: ["docs/**"]
```

This fires only when the push changes at least one file under docs/. The pattern ** matches any number of folder levels. Teams use this to keep a slow test suite from running when someone only fixed a typo in the documentation, or the other way around, to rebuild the docs site only when docs change.

Filters combine. Both must match for the workflow to run:

One YAML detail: a pattern like docs/** starts with a plain letter, but patterns beginning with * must be quoted, because * is a YAML syntax character. Quoting every pattern is a safe habit.

### schedule and workflow_dispatch
In this lesson you will learn two triggers that do not need a code change: schedule runs a workflow on a timer, and workflow_dispatch adds a manual run button.

#### schedule: run on a timer
Some jobs should run whether or not anyone pushed: a nightly full test run, a weekly dependency check, a daily backup. The schedule event uses cron syntax, a compact five-field time format:
```
on:
  schedule:
    - cron: "0 2 * * *"
```

The five fields, left to right: minute, hour, day of month, month, day of week. A * means every. So 0 2 * * * reads "minute 0, hour 2, every day" - 02:00 every night, in UTC time.

More examples: 30 6 * * 1 is 06:30 every Monday (day of week 1). 0 */6 * * * is every 6 hours. Always quote the cron string: it contains *, a YAML syntax character.

#### workflow_dispatch: a run button
```
on:
  workflow_dispatch:
```

This adds a "Run workflow" button on the workflow's page in GitHub. Nothing runs until a person clicks it. Teams use it for actions that need a human decision, like the continuous delivery deploy from topic 01, and for testing a workflow without pushing a fake commit.

Note the trailing colon with nothing after it: the event takes settings later (like input fields), so it is written as a map key even when empty.

A workflow can combine all of these. A nightly run plus a manual button is a very common pair, and you will build exactly that in this topic's last task.

## 5.run steps and uses steps
In this lesson you will learn the two kinds of steps a job can contain, and how a job executes them.

Back to pixelpost. The team's release checklist has entries like "run the tests" and "download the code first". When you turn that checklist into a workflow, each entry becomes a step, and the steps live inside a job. There are exactly two kinds:

- A run step executes a shell command you write yourself.
- A uses step executes an action, a ready-made piece of work someone else packaged. You will meet actions properly in the next topic.
```
jobs:
  checks:
    runs-on: ubuntu-latest
    steps:
      - name: Say hello
        run: echo "hello from pixelpost"
      - name: Get the code
        uses: actions/checkout@v4
```
Each step starts with a dash, because steps: is a list. The name: is optional but always worth writing: it is the label you will see in the logs when something fails.

#### Steps run in order, top to bottom
A job executes its steps one at a time, in the order they appear in the file. Step 2 does not start until step 1 finishes. If a step fails, the job stops and the remaining steps are skipped.

Order matters because the steps share one machine. Whatever step 1 leaves on disk, step 2 can read. That is the whole reason a job exists: it is a group of steps that need each other's results on the same machine.

### Multi-line run, shell, and working-directory
In this lesson you will learn how to put several commands in one step, how to choose the shell, and how to change the folder a step runs in.

#### Several commands in one run step
A single command fits after run: on the same line. For several commands, put a pipe character after run: and indent each command on its own line:
```
- name: Prepare the report folder
  run: |
    mkdir -p report
    date > report/generated-at.txt
    echo "report folder ready"
```

#### Choosing the shell
On a Linux runner the commands run in bash by default. The shell: key makes the choice explicit, or switches to another shell:
```
- name: Run with bash
  shell: bash
  run: echo "running in $0"
```

You rarely need shell: on Linux, but you will see it often in real workflows, especially ones that also run on Windows, where the default is PowerShell instead.

#### Changing the folder with working-directory
Every run step starts in the root of the workspace. If the commands belong in a subfolder, set working-directory: instead of writing cd at the top of the script:
```
- name: Work inside the app folder
  working-directory: app
  run: ls
```
This keeps the run script about the work, not about navigation, and it only affects that one step.

## 6.What is an action
In this lesson you will learn what an action is and how the uses: line names one.

The pixelpost team needs their workflow to download the code, install Python, and later upload a build result. None of that is special to pixelpost. Thousands of teams need the exact same steps, so people packaged them as reusable units called actions. A uses step pulls one in:
``
- name: Get the code
  uses: actions/checkout@v4
```

This example uses an action stored in a GitHub repository. Its uses: value has three parts:

- owner: the GitHub account that publishes the action. The actions account belongs to GitHub itself.
- repository: the action's repo name.
- ref: a tag, branch, or commit SHA after the @ sign. Here, v4 is a tag.
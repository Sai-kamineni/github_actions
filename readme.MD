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
```
- name: Get the code
  uses: actions/checkout@v4
```

This example uses an action stored in a GitHub repository. Its uses: value has three parts:

- owner: the GitHub account that publishes the action. The actions account belongs to GitHub itself.
- repository: the action's repo name.
- ref: a tag, branch, or commit SHA after the @ sign. Here, v4 is a tag.
When the job reaches this step, the runner downloads that repository at that ref and executes the code inside it. You write one line; the action's authors wrote and maintain the rest.

Actions can also live in the same repository through a local path, or come from a Docker image. Those forms use different uses: addresses; owner/repository@ref names a remote repository action.

#### The marketplace
Published actions are listed in the GitHub Marketplace at github.com/marketplace. There are actions for checking out code, setting up language toolchains, sending Slack messages, publishing releases, and much more. Before writing a long run script for a common task, it is normal to check whether a well maintained action already does it.

Anyone can publish an action, so the owner matters. Actions under the actions owner are maintained by GitHub and are the safe default for the basics.

### actions/checkout and the empty runner
In this lesson you will learn why runners start with no code on them, and how actions/checkout fixes that.

Here is a mistake every beginner makes once. The pixelpost team writes a workflow with one step, run: cat README.md, and it fails: no such file or directory. The repository clearly has a README. So where did it go?

Nowhere. It was never there. The runner is a fresh virtual machine created just for this job. It has an operating system and common tools installed, but not a single byte of your repository. GitHub triggers the workflow because of your push, but it does not copy the code onto the runner for you.

#### The fix is one uses step
The actions/checkout action clones your repository into the runner's workspace. It reads the event that triggered the run, so it checks out exactly the commit that was pushed:
```
steps:
  - name: Get the code
    uses: actions/checkout@v4
  - name: Read the readme
    run: cat README.md
```

Order matters, as you learned in the previous topic: checkout must come before any step that reads repository files. That is why almost every workflow you will ever read has checkout as its first step.

One useful flip side: if a job never touches the repository files, for example it only calls an external API, it does not need checkout at all.

## 7.How this VM talks to GitHub
In this lesson you will learn what a personal access token is and how this machine will use it for everything that touches GitHub.

Everything you built so far lived on this VM. From this topic on, your workflows run on GitHub's servers, inside your own GitHub account. That means this machine needs a way to prove to GitHub that it is you.

You will need a GitHub account. If you do not have one, create a free account at github.com before the next lesson.

### Passwords do not work here
GitHub does not accept your account password from tools like git or curl. Instead it uses personal access tokens. A token is a long random string that you generate in your GitHub settings. It acts like a password with two extra properties:

Scopes. A token only has the permissions you grant it. Ours will get repo (read and write your repositories) and workflow (update workflow files). Nothing more.
Revocable. If a token leaks, you delete that one token. Your account and other tokens stay safe.

#### One token, three uses
You will store the token in one file on this VM, and everything reads it from there:

The checks in this skill grade real state on GitHub. When a task says "the workflow ran successfully", the check calls the GitHub API with your token and looks at the actual run. There is nothing to paste and nothing to self-report.

Treat the token like a password. Never commit it to a repository and never paste it into a workflow file. You will store it in one file with tight permissions, and that is the only place it lives.

## 8.Your first workflow
### From push to run
In this lesson you will learn what happens between pushing a workflow file and seeing it run on GitHub.

So far you wrote workflow files on this VM and checked them with actionlint. Nothing executed. That is because a workflow file is only instructions. Something has to read those instructions and act on them, and that something lives on GitHub's side.

#### The chain of events
When you push a commit to GitHub, this happens:

- GitHub receives the push and looks in .github/workflows/ on the pushed branch.
- For every workflow file whose on: section matches the event (a push, in this case), GitHub creates a run.
- Each job in the run is handed to a runner. A GitHub-hosted runner uses a fresh virtual machine for this job.
- The runner executes the steps in order and streams logs back. GitHub discards a GitHub-hosted runner after the job ends; self-hosted runners can persist.

Two details matter here. First, the workflow file must be on the branch you push, inside .github/workflows/, or GitHub never sees it. Second, GitHub-hosted runners start fresh. Jobs that need files from the repository must explicitly check out the code; checkout is not automatic.

#### Where results live
Every run, its jobs, and its logs are stored by GitHub. You can see them in the repository's Actions tab in the browser, and you can read the same data from the GitHub API with curl, the way the checks in this course do.

### Anatomy of a run
In this lesson you will learn how a run is structured and how to read one from the API, the same way this course's checks do.

Your push just created a run. On GitHub, a run is a tree with three levels:

- Run: one execution of one workflow, triggered by one event. It has a number, a status, and a conclusion.
- Jobs: the parallel units inside the run. Each GitHub-hosted job gets a fresh runner VM; self-hosted - jobs are assigned to available registered runners.
Steps: the ordered commands inside a job. Each step has its own log and its own success or failure.

#### Status versus conclusion
A run has two fields that beginners mix up:

- status says where the run is in its lifecycle. Common values include queued, in_progress, and completed; waiting or pending states can also appear.
- conclusion says how it ended, and is empty until the run completes. Common conclusions include success, failure, cancelled, and skipped; other conclusions are possible.

A run can be completed and still be a failure. Always read both.

#### Reading runs with curl
The Actions tab shows all of this in the browser. The same data comes from the API:
```
curl -s -H "Authorization: token $(cat /root/.github/token)" \
  "https://api.github.com/repos/OWNER/actions-journey/actions/runs?per_page=1" \
  | jq '.workflow_runs[0] | {status, conclusion, name}'
```

This exact pattern, list the runs and inspect the newest one, is how every GitHub-side check in this course grades your work.

## 9.Logs and Debugging
### How a run fails
In this lesson you will learn what a failed run is made of, so you know exactly where to look when one goes red.

Sooner or later a pixelpost push turns the Actions tab red. The run says failure, but the run did not fail as one big block. A common command failure starts at the bottom of the tree you met in the run anatomy lesson, and climbs up:

- A step runs a command, and the command exits with a non-zero exit code. That step's conclusion becomes failure.
- By default, the job stops there. Ordinary remaining steps are skipped, and the job's conclusion becomes failure.
- A failed required job makes the run fail too.

This is the default path. continue-on-error can allow a step or job failure, while conditions such as if: always() or if: failure() can run later steps. Actions can also fail during setup or execution without a learner-written shell command.

Debugging is walking this chain in reverse: the red run points you to the failed job, the failed job points you to the failed step, and the failed step's log shows the command and its error message. The message at the bottom of that log is the actual problem. Everything above it is just where the problem surfaced.

#### The two failures you will see most
Almost every early workflow failure is one of these:

- Command not found. The command name has a typo, or the tool is not installed on the runner.
- No such file or directory. The step reads a repository file, but no checkout step ran before it, so the workspace is empty. You saw why in topic 06.

Both look dramatic in a red run and both are one-line fixes once you find the failed step.

### Reading a run with the jobs API
In this lesson you will learn how to find the failed step of a run from the command line, using the jobs API.

In the run anatomy lesson you listed runs with curl and read their status and conclusion. That tells you a run failed, but not where. The next level down has its own endpoint. First, grab the id of the newest run of one workflow:
```
curl -s -H "Authorization: token $(cat /root/.github/token)" \
  "https://api.github.com/repos/OWNER/actions-journey/actions/workflows/hello.yml/runs?per_page=1" \
  | jq '.workflow_runs[0].id'
```
Then ask for that run's jobs, putting the id into the URL:
```
curl -s -H "Authorization: token $(cat /root/.github/token)" \
  "https://api.github.com/repos/OWNER/actions-journey/actions/runs/RUN_ID/jobs" \
  | jq '.jobs[] as $job | $job.steps[] | {job: $job.name, name, conclusion}'
```
This endpoint is paginated and returns 30 jobs by default. For runs within that first page, each job includes its step list with a name and conclusion per step. The jq filter keeps the job name while flattening those steps into a line-by-line report:
```
{ "job": "checks", "name": "Set up job", "conclusion": "success" }
{ "job": "checks", "name": "Read the readme", "conclusion": "failure" }
{ "job": "checks", "name": "Show the Python version", "conclusion": "skipped" }
```
Read each job separately. Within one ordinary job, its first failed step is where that job's problem surfaced; later ordinary steps may be skipped. Parallel jobs have independent timelines, and conditions can allow later steps to run. Steps like "Set up job" are added by GitHub around your own steps; inspect them too when they fail.

Two habits make this workable:
- Save the response to a file with > so you can run several jq filters over it without calling the API again.
- While a run is still progressing, unfinished jobs or steps can have null conclusions even though completed steps already have one.

## 10.What is a context
In this lesson you will learn what a context is and how the ${{ }} syntax reads values from one.

Every workflow you pushed so far did the same thing on every run. Real pipelines are not like that. A deploy job needs to know which branch triggered it. A notification step wants the commit id it just built. That information changes on every run, so you cannot type it into the workflow file.

GitHub solves this with contexts. A context is a bundle of named values available as the run and its jobs progress. Your workflow file says which value it wants, and GitHub evaluates the expression when that workflow field is processed.

#### The expression syntax
You ask for a context value with an expression. An expression is wrapped in ${{ }}:
```
steps:
  - name: Show the branch
    run: echo "This run was triggered on ${{ github.ref }}"
```
GitHub reads the workflow file, sees ${{ github.ref }}, and replaces it with the real Git ref of this run, which can identify a branch, tag, or pull-request merge ref. By the time the shell runs the echo, the expression is gone and only the value remains.

#### The main contexts
There are several contexts. The three you will meet first:

- github - facts about the run and the repository: branch, commit, event, who pushed.
- runner - facts about the machine executing the job, like its operating system.
- job - the status of the current job while it runs.

Property syntax commonly reads a context value as context.field, for example github.sha or runner.os. Index syntax such as github['sha'] also works, and functions can receive a whole context object, as in toJSON(github).

### The github context
In this lesson you will learn the fields of the github context you will use in almost every real workflow.

The github context describes the event that started the run. When a push triggers a workflow, GitHub fills the context with the branch, the commit, and the person who pushed. These are the fields worth memorizing:

- github.repository - owner and repository name, like priya/actions-journey.
- github.ref - the full reference that was pushed, like refs/heads/main.
- github.ref_name - just the short branch name, like main.
- github.sha - the full commit id the run is building.
- github.event_name - the event that started the run, like push.
- github.actor - the username that triggered the run.

### Reading them in a step
A step can print any of these:
```
steps:
  - name: Describe this run
    run: |
      echo "Repository: ${{ github.repository }}"
      echo "Branch: ${{ github.ref_name }}"
      echo "Commit: ${{ github.sha }}"
      echo "Event: ${{ github.event_name }}"
      echo "Triggered by: ${{ github.actor }}"\
```

Push this, open the run log, and every line shows a real value. The same workflow file prints different values on every push, because the context changes with each run.

### The runner context
The runner context works the same way but describes the machine, not the event. The two common fields:

runner.os - the operating system, like Linux.
runner.arch - the processor type, like X64.

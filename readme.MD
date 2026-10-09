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

These matter later, when one workflow runs on several operating systems and a step needs to know which machine it landed on.

### Expression functions
In this lesson you will learn three functions you can call inside ${{ }} expressions: contains, startsWith, and format.

Expressions can do more than read a single value. GitHub gives you a small set of built-in functions to test and combine values before a step sees them.

#### contains and startsWith
Both answer a yes or no question about a value:
```
${{ contains(github.event.head_commit.message, 'urgent') }}
${{ startsWith(github.ref, 'refs/heads/release') }}
```
- contains(a, b) is true when b appears anywhere inside a. The example above is true when the commit message has the word urgent in it.
- startsWith(a, b) is true when a begins with b. The example is true for any branch whose name starts with release.

Both are case-insensitive. Their real power appears in a later topic, where a job can be skipped unless an expression is true. For now you can print their result to see how they behave.

### format
format builds one string from a template and values. The template uses {0}, {1}, and so on as slots:
```
${{ format('build {0} of {1}', github.run_number, github.ref_name) }}
```

On the fifth push to main this produces build 5 of main. github.run_number is a counter GitHub increases by one for every run of a workflow, which makes it useful for labels and version strings.

Teams use format to build release names, image tags, and messages that carry the run's identity.

## 11.Variables and env

In this lesson you will learn how to set environment variables in a workflow and what happens when the same variable is set at more than one level.

Steps run shell commands, and shell commands read environment variables. A workflow can define those variables with an env block. The block can sit at three levels:
```
name: env-demo
on: push
env:
  APP_NAME: journey        # workflow level: every job sees it
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      LOG_LEVEL: info      # job level: every step in this job
    steps:
      - name: Verbose step
        env:
          LOG_LEVEL: debug # step level: this step only
        run: echo "$APP_NAME with log level $LOG_LEVEL"
```
Inside run commands you read them like any shell variable, with $APP_NAME. No ${{ }} needed - the runner exports them before the shell starts. Variable names stay exact: APP_NAME and LOG_LEVEL include their underscores.

#### Precedence: the closest level wins
### env at three levels
When the same name is set at more than one level, the most specific one wins. Step beats job, job beats workflow.

In the example, the verbose step prints journey with log level debug. Every other step in the job would print info, because only this step overrides it.

This pattern is everywhere in real pipelines: sensible defaults at the workflow level, one job that needs something different, one step that needs even more detail.

### Repository variables
In this lesson you will learn what repository variables are and how to create and list them with the GitHub API.

The env blocks from the last lesson live inside the workflow file. Change a value, and you must edit the file and push a commit. Some values do not belong there: a deploy region, a server name, a flag the team flips without touching code.

For these, GitHub stores repository variables on the repository itself, outside any file. Every workflow in the repository can read them through the vars context:
```
steps:
  - name: Show the target region
    run: echo "Deploying to ${{ vars.DEPLOY_REGION }}"
```

Change the variable's value, and the next run picks it up. No commit needed.

#### Creating a variable with curl
Repository variables have their own API endpoint. One POST request creates a variable:
```
login=$(curl -s -H "Authorization: token $(cat /root/.github/token)" \
  https://api.github.com/user | jq -r '.login // empty')
curl -s -H "Authorization: token $(cat /root/.github/token)" \
  -d '{"name":"DEPLOY_REGION","value":"eu-west-1"}' \
  "https://api.github.com/repos/$login/actions-journey/actions/variables"
```

The JSON body carries two fields: the variable's name and its value. A successful create returns nothing, which is normal for this endpoint.

#### Listing variables
A GET request on the same endpoint lists what exists:
```
curl -s -H "Authorization: token $(cat /root/.github/token)" \
  "https://api.github.com/repos/$login/actions-journey/actions/variables" | jq .
```

The response shows each variable's name and value. That last part matters: variables are visible to anyone who can read the repository's settings. They are for configuration, never for passwords. The next topic covers what to use for those.

## 12.What is a secret
In this lesson you will learn what a repository secret is and how it differs from a repository variable.

Pipelines constantly need sensitive values: an API token to push a package, a password to reach a database, a key to log in to a server. The repository variables from the last topic are the wrong place for these. Anyone who can list the variables sees their values in plain text.

For sensitive values, GitHub has repository secrets. A secret looks like a variable - a name and a value stored on the repository - but GitHub treats the value very differently:

- The value is encrypted the moment you save it.
- GitHub never shows it again. Not in the settings page, not in the API. You can replace a secret or delete it, but you cannot read it back.
- Only a running workflow can receive the decrypted value.

#### What the API shows
The secrets endpoint of the API lists names and dates, nothing more:
```
login=$(curl -s -H "Authorization: token $(cat /root/.github/token)" \
  https://api.github.com/user | jq -r '.login // empty')
curl -s -H "Authorization: token $(cat /root/.github/token)" \
  "https://api.github.com/repos/$login/actions-journey/actions/secrets" | jq .
```

There is no value field in the response. This is a deliberate design: if a token leaked and could read your secrets back, one stolen token would expose every password the repository holds. Because values are write-only, a leaked token cannot harvest them.

The simple rule: configuration goes in variables, anything you would not paste in a chat message goes in secrets.

### Secrets in workflows and log masking
In this lesson you will learn how a workflow reads a secret and how GitHub keeps it out of the logs.

A workflow reads secrets through the secrets context. The safe pattern is to hand the value to a step as an environment variable:
```
steps:
  - name: Call the API
    env:
      APP_TOKEN: ${{ secrets.APP_TOKEN }}
    run: |
      if [ -n "$APP_TOKEN" ]; then
        echo "token is available to this step"
      fi
```

The step's shell sees $APP_TOKEN like any environment variable, but the value never appears in the workflow file. Keep the underscore: APP_TOKEN is the exact variable name.

#### Masking
Run logs are visible to everyone who can see the repository, so a secret printed in a log would be a leak. GitHub defends against accidents with masking: whenever a known secret value appears in log output, it is replaced with *** before the log is stored.

Masking is a safety net, not permission. Never write a step that echoes a secret on purpose. Masking only matches the exact value: a secret that gets transformed first - encoded or split in half, for example - can slip through unmasked. Writing the unchanged value to a file does not itself defeat masking if another step later prints that exact known value.

Two habits keep you safe:
- Pass secrets to commands through env, never paste them into run lines or commit them in any file.
- Treat any secret that ever appeared in output, even masked, as worth rotating if you are unsure.

## 13.Job dependencies
### Ordering jobs with needs
In this lesson you will learn how to make one job wait for another, using the needs keyword.

Back in topic 05 you saw that independent jobs are eligible to run in parallel. GitHub-hosted jobs receive fresh runners, while self-hosted jobs use available registered runners. But think about a real pipeline for your actions-journey repository: you want to run the tests first, and only deploy if the tests pass. Starting both at the same time would be wrong. A deploy that does not wait for tests is not a pipeline, it is a race.

#### The needs keyword
needs on a job lists the jobs that must finish successfully before this job starts:
```
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - run: echo "running tests"
  deploy:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - run: echo "deploying"
```

Now deploy sits in a waiting state until test finishes. Two rules to remember:

- By default, if test fails, deploy is skipped. A job-level condition such as if: ${{ always() }} can explicitly allow a dependent job to run after a failed or skipped need.
- A job can need several jobs: needs: [test, lint] means wait for both.

#### The graph this creates
With needs, your jobs form a graph. GitHub reads it and runs everything that can run, as early as it can:

Jobs without needs still start immediately and in parallel. Only the jobs you connect wait for each other.

### Passing data with job outputs
In this lesson you will learn how one job hands a value to a job that needs it.

needs gives you order, but jobs still run on separate runners. Imagine a build job that computes a version number like 1.4.2. The deploy job runs on a different machine, so that number does not exist there. Files, variables, everything from the build runner is gone. You need a way to pass the value across.

#### Step outputs first
A step publishes a value by writing a name=value line into the file that the GITHUB_OUTPUT environment variable points to:
```
steps:
  - name: Compute version
    id: ver
    run: echo "version=1.4.2" >> "$GITHUB_OUTPUT"
```

The step needs an id so other places can refer to it. Inside the same job, another step reads it with the expression steps.ver.outputs.version.

#### Lifting a step output to a job output
To make the value visible outside the job, declare it in the job's outputs: map:
```
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.ver.outputs.version }}
    steps:
      - name: Compute version
        id: ver
        run: echo "version=1.4.2" >> "$GITHUB_OUTPUT"
  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - run: echo "Deploying ${{ needs.build.outputs.version }}"
```
The chain has three links: the step writes to $GITHUB_OUTPUT, the job's outputs: map lifts the step output up, and the downstream job reads it through the needs context. The downstream job must list the producer in its own needs:, otherwise the needs context has nothing in it.

### Fan-out and fan-in
In this lesson you will learn the two shapes that most real pipelines are built from: fan-out and fan-in.

A straight chain like build, then test, then deploy works, but it wastes time. If linting and unit tests do not depend on each other, why should the tests wait for the linter? Runners are separate machines. Let them work at the same time.

#### Fan-out: one job unlocks many
Fan-out means several jobs all need the same earlier job. The moment that job finishes, the whole group starts in parallel:

In YAML this is just three jobs that each say needs: build.

#### Fan-in: many jobs unlock one
Fan-in is the mirror image. One job lists several jobs in its needs, and starts only when all of them have succeeded:

Here package declares needs: [lint, unit-tests]. If either one fails, package is skipped, exactly like in the first lesson.

#### Real pipelines combine both
A common shape is a diamond: build fans out into parallel checks, and a release job fans them back in:
Total time drops to the slowest path through the graph, not the sum of all jobs.


## 14.Conditions
### The if keyword
In this lesson you will learn how to make a step or a whole job run only when a condition is true.

Think about the pipeline you built in the last topic. It runs the deploy job on every push, including pushes to experiment branches. On a real team that would be a problem: you want tests everywhere, but deploys only from main. The if keyword is how you say that.

#### if on a step
An if on a step holds an expression, built from the contexts you met in topic 10. The step runs only when the expression is true:
```
steps:
  - name: Deploy
    if: github.ref == 'refs/heads/main'
    run: echo "deploying"
```
On a push to any other branch, GitHub marks the step as skipped. It shows up grey in the log, costs nothing, and the job carries on with the next step.

Inside if you can usually skip the ${{ }} wrapper - GitHub treats the whole line as an expression already. An expression beginning with ! must use the wrapper or be quoted so YAML does not treat it as a tag. Combine conditions with &&, ||, and !, just like in a shell:
`if: github.ref == 'refs/heads/main' && github.event_name == 'push'`

#### if on a job
The same keyword works one level up. A skipped job never starts a runner at all:
```
jobs:
  deploy:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: echo "deploying"
```

One thing to know: by default, when a job is skipped, jobs that needs it are skipped too. A downstream job can override this with a condition such as if: ${{ always() }}.

### Status functions
In this lesson you will learn the functions that let a step react to how the run is going: success(), failure(), and always().

Here is the default rule first, because everything builds on it: when a step fails, GitHub skips every later step in that job. Usually that is what you want. But not always. Imagine a job that starts a test database, runs the tests, and then has to stop the database. If the tests fail, the stop step is skipped, and the database keeps running until the runner dies. Cleanup work must not depend on success.

### The three functions
You use them inside a step's if:

- success() - true when no earlier step in the job has failed. This is the invisible default: a step with no if behaves as if: success().
- failure() - true when some earlier step has failed. Use it for reactions to a broken run, like printing extra debug information.
- always() - true no matter what happened. Use it for cleanup that must run in both the good and the bad case.

The cleanup pattern
```
steps:
  - name: Start test database
    run: echo "db up"
  - name: Run tests
    run: ./run-tests.sh
  - name: Collect debug info
    if: failure()
    run: echo "dumping logs for the red run"
  - name: Stop test database
    if: always()
    run: echo "db down"
```

If the tests pass, the debug step is skipped and cleanup runs. If the tests fail, the debug step runs, cleanup still runs, and the job is still marked failed. always() does not rescue the job's conclusion - it only rescues that one step from being skipped.

You can also mix status functions with normal conditions, for example if: failure() && github.ref == 'refs/heads/main' to react to failures on main only.

## 15.Matrix
### What a matrix is
In this lesson you will learn how one job definition can run as many jobs, once per combination of values.

Say your team supports three Python versions. Users run 3.10, 3.11, and 3.12, so the tests must pass on all three. You could copy the test job three times and change one line in each copy. It works, until someone updates two copies and forgets the third. The matrix keyword removes the copies.

### strategy.matrix
```
jobs:
  tests:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]
    steps:
      - uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
      - name: Run a version check
        run: python3 -c "import sys; print(sys.version)"
```
GitHub expands this one definition into three separate jobs, named after the values: tests (3.10), tests (3.11), and tests (3.12). Each runs on its own fresh runner, in parallel, like any other jobs.

Two new pieces here:
- The matrix context: matrix.python-version holds this job's own value from the expansion, so every copy installs a different Python.
-  actions/setup-python@v5: a ready-made action, like the checkout action from topic 06, that installs the requested Python version on the runner.

#### More than one axis
A matrix can have several lists, and GitHub builds every combination:
```
matrix:
  python-version: ["3.11", "3.12"]
  os: [ubuntu-latest, macos-latest]
```

Two versions times two systems is four jobs. Set runs-on: ${{ matrix.os }} and each job lands on the right machine. Three axes multiply again, so counts grow fast: keep matrices as small as your real support promise.

### include, exclude, and fail-fast
In this lesson you will learn how to adjust a matrix after expansion, and how one failing cell affects the rest.

A pure grid is often almost right. Your team tests two Python versions on two systems, but the macOS build for the oldest Python is broken upstream, and you also want one extra job that tries the next Python beta. The matrix has keywords for both cases.

#### exclude: remove combinations
```
strategy:
  matrix:
    python-version: ["3.11", "3.12"]
    os: [ubuntu-latest, macos-latest]
    exclude:
      - python-version: "3.11"
        os: macos-latest
```

The grid expands to four, then the matching combination is removed, leaving three jobs.

#### include: add combinations
```
strategy:
  matrix:
    python-version: ["3.11", "3.12"]
    include:
      - python-version: "3.13"
```

include bolts extra jobs onto the grid. It can also add an extra key to an existing combination, for example giving just one cell an environment flag.

#### fail-fast
By default, fail-fast is true: the moment one matrix job fails, GitHub cancels all the still-running sisters. That saves minutes, but it hides information - you learn that 3.10 is broken and nothing about 3.11 and 3.12. Turn it off to let every cell finish:
```
strategy:
  fail-fast: false
  matrix:
    python-version: ["3.10", "3.11", "3.12"]
```

Whether the default is right depends on why the matrix exists, and you will weigh exactly that trade in the scenario after the next task.

## 16.Artifacts
### What is an artifact
In this lesson you will learn what an artifact is, how the upload-artifact action saves one, and how long GitHub keeps it.

Every job using a GitHub-hosted runner gets a fresh runner, which GitHub discards when the job ends. Whatever the job wrote to disk - a built package, a test report, a log file - is discarded with it. Usually that is fine. But sometimes the file IS the point. A build job that produces a package nobody can download did not really produce anything.

An artifact is a file (or a folder of files) that a workflow hands to GitHub for safekeeping before the runner dies. GitHub stores it next to the run, and you can download it from the run page or through the API.

Uploading with actions/upload-artifact
```
- name: Upload the report
  uses: actions/upload-artifact@v4
  with:
    name: build-report
    path: report.txt
```

#### Two inputs matter:
- name is the label the artifact appears under on the run page. Pick something you would search for later.
- path is what to save. It can be one file, a folder, or a glob like dist/*.txt.

### Retention
Artifacts are not kept forever. The default is 90 days, then GitHub deletes them. You can shorten that per artifact:
```
with:
  name: build-report
  path: report.txt
  retention-days: 5
```
Shorter retention is a good habit for throwaway files like test reports. Storage on private repositories counts against a quota, and even on public ones a pile of stale artifacts is just noise.

### Passing files between jobs
In this lesson you will learn how one job hands a file to another job using download-artifact.

You know from the job dependencies topic that jobs run on separate runners, and that needs: controls their order. Order is not enough, though. A build job and a test job may run in the right sequence, but the test job's runner starts empty. The package the build job created is on a machine that no longer exists.

Artifacts solve this. The first job uploads the file, GitHub stores it, and the second job downloads it.

#### Downloading with actions/download-artifact
```
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build the package
        run: echo "the package" > app.txt
      - name: Upload the package
        uses: actions/upload-artifact@v4
        with:
          name: app-package
          path: app.txt
  verify:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download the package
        uses: actions/download-artifact@v4
        with:
          name: app-package
      - name: Check it arrived
        run: test -f app.txt
```

Three details to notice:
- needs: build is required. Without it the two jobs run at the same time, and the download step would look for an artifact that does not exist yet.
- The name in download-artifact must match the name used in upload-artifact, character for character.
- The files land in the job's working directory by default. A path input can place them somewhere else.

This upload-then-download pattern is how real pipelines split work: build once, then test, scan, or deploy the same files in later jobs.

## 17.cache
### What the cache action does
In this lesson you will learn what actions/cache is for and how it differs from an artifact.

Every runner starts empty. That kept things clean in earlier topics, but it has a price: work gets repeated. A Python job downloads the same packages from the internet on every single run. The packages did not change; the runner just forgot them.

A cache saves a folder at the end of one run and restores it at the start of the next. Same repository, different runs. The download step still runs, but it finds everything already on disk and finishes in seconds.

#### The cache step
```
- name: Cache pip downloads
  uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: pip-v1
```
- path is the folder to save and restore. For pip that is ~/.cache/pip, where pip keeps downloaded packages.
- key is the label the saved folder is stored under. A run that asks for the same key gets the same folder back.

The step does both directions. At its position in the job it restores the folder if the key exists (a cache hit). If not (a cache miss), the job continues with nothing, and at the end of a successful job the folder is saved under that key.

#### Cache is not an artifact
The two features store files but solve different problems. An artifact is an output you want to keep and download: a package, a report. A cache is a speed-up you could throw away at any time: if GitHub deleted every cache tonight, your workflows would still pass, just slower. GitHub does exactly that to caches not used for 7 days.

### Cache keys and restore-keys
In this lesson you will learn how to build a cache key that changes when it should, and how restore-keys give you a partial hit instead of a miss.

A fixed key like pip-v1 has a problem: it never changes. Add a new package to requirements.txt and the workflow keeps restoring the old folder, then saves nothing new, because the key already exists. The cache is stale forever.

The fix is to build the key from the file that defines your dependencies:
```
- name: Cache pip downloads
  uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: pip-${{ hashFiles('requirements.txt') }}
    restore-keys: |
      pip-
```

hashFiles is an expression function that computes a checksum of the named file. Same file contents, same key, cache hit. Edit requirements.txt and the checksum changes, so the key is new and the old cache no longer matches.

#### restore-keys: a useful second choice
A new key means a miss, and a miss means downloading everything again - even though only one package changed. restore-keys is a list of prefixes to try when the exact key misses. pip- matches the most recent cache whose key starts with pip-, so the job starts with yesterday's folder, downloads only the one new package, and saves the result under the new exact key.

One warning: after a restore-keys match, the job runs with an older folder than the key describes. For a pip download cache that is harmless, pip checks versions anyway. For folders where stale files can poison the result (like some build outputs), an exact key with no restore-keys is the safer choice.

## 18.CI for a real application
### Meet the notekeeper application
In this lesson you will meet the application this topic builds a CI pipeline for. You will not write any application code - just get oriented.

So far your workflows echoed text and installed packages. Real CI protects a real codebase. For that, this topic uses notekeeper, a small notes tool written in Python. It stores notes in a JSON file and supports adding, listing, searching, and deleting them.

#### The files
The app lands in /root/actions-pipelines in the next task. It has four files:

- notes.py - the whole application: load and save the JSON file, plus one function per command.
- test_notes.py - five pytest tests covering adding, deleting, and searching notes.
- requirements.txt - one pinned dependency, pytest. The app itself uses only the Python standard library.
- README.md - usage and how to run the tests.

### How the tests run
pytest discovers test_notes.py automatically and calls the functions in notes.py directly. Locally, a developer runs:
```
python3 -m pip install -r requirements.txt
python3 -m pytest
```
Five tests, well under a second. That exact pair of commands is what your CI pipeline will run on every push: install what requirements.txt names, then run pytest. If any test fails, the run goes red.

### A CI workflow for Python
In this lesson you will learn the standard shape of a Python CI workflow: pick a Python version, install dependencies with caching, run the tests.

A CI job for notekeeper has to do on the runner exactly what a developer does locally: install pytest, run pytest. Two pieces are new.

#### setup-python
Runners come with some Python preinstalled, but serious pipelines pin the version so a runner image update cannot silently change behavior. actions/setup-python does that, and it has caching built in:
```
- name: Set up Python
  uses: actions/setup-python@v5
  with:
    python-version: "3.12"
    cache: pip
```
 
The cache: pip input is the pattern from the caching topic, packaged: it caches pip's download folder with a key built from hashing requirements.txt. One line instead of a whole actions/cache step.

#### Trigger on push and pull_request
CI for an application should also run when someone opens a pull request, so broken changes are caught before merge:
```
on: [push, pull_request]
```

#### The whole job
```
name: ci
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip
      - name: Install dependencies
        run: pip install -r requirements.txt
      - name: Run tests
        run: python3 -m pytest
```

Read it as a sentence: get the code, get Python, get the dependencies, prove the code works. Nearly every Python CI pipeline you will meet is this sequence with extras bolted on.

## 19.Reusable workflows
In this lesson you will learn what a reusable workflow is, the trigger that makes one, and the uses: line that calls it.

The pixelpost team now has three workflows: one for the app, one for the docs site, one for a small internal tool. All three contain the same twenty lines: check out the code, set up Python, install dependencies, run pytest. When the team upgrades Python, someone edits the same lines three times. One day they miss a file, and one pipeline quietly tests the wrong version.

A reusable workflow fixes this. You write the shared steps once, in their own workflow file, and other workflows call it like a function.

#### The workflow_call trigger
A workflow becomes callable by listing workflow_call in its on: section:
```
name: checks
on:
  workflow_call:
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "running the shared checks"
```
With only this trigger, the workflow never starts on its own. No push, no schedule. It waits until another workflow calls it.

#### Calling it
The caller replaces a whole job with a uses: line:
```
name: pipeline
on: push
jobs:
  checks:
    uses: ./.github/workflows/checks.yml
```

Note what is missing: the calling job has no runs-on: and no steps:. It hands everything to the called workflow, which brings its own jobs and runners.

The uses: path comes in two forms:

- Same repository: ./.github/workflows/checks.yml - a path starting with ./, always from the repository root.
- Another repository: owner/repo/.github/workflows/checks.yml@main
and the @ref (a branch, tag, or commit) is required here.

In the run view, jobs from the called workflow appear under the caller's job name, joined with a slash, like checks / test. The jobs API shows the same names, which is how this topic's checks will confirm your call worked.

### Passing inputs and secrets
In this lesson you will learn how a caller sends values into a reusable workflow with with: and secrets:.

A shared checks workflow is only useful if each caller can adjust it a little. The pixelpost app needs Python 3.12, the internal tool still needs 3.11. Same steps, one different value. That is what inputs are for.

#### Declaring inputs
The reusable workflow declares its inputs under workflow_call:
```
on:
  workflow_call:
    inputs:
      app-name:
        type: string
        required: true
```

Each input has a type (string, number, or boolean) and can be required or carry a default. Inside the reusable workflow you read it through the inputs context:
```
steps:
  - run: echo "Checking ${{ inputs.app-name }}"
```

#### The caller fills them in
The caller passes values with with:, right under uses::
```
jobs:
  checks:
    uses: ./.github/workflows/checks.yml
    with:
      app-name: pixelpost
```

#### Secrets do not flow automatically
Repository secrets are not visible inside a called workflow by default. The reusable workflow must declare which secrets it expects:
```
on:
  workflow_call:
    secrets:
      NOTIFY_TOKEN:
        required: true
```
And the caller must hand them over:
```
jobs:
  checks:
    uses: ./.github/workflows/checks.yml
    secrets:
      NOTIFY_TOKEN: ${{ secrets.NOTIFY_TOKEN }}
```
There is a shortcut, secrets: inherit, which passes every secret the caller can see. It is convenient inside one repository, but listing secrets by name documents exactly what the shared workflow touches, so this course uses explicit names.

One boundary to remember: a reusable workflow replaces a whole job. You cannot call one from inside a step. When you want a shareable block of steps, you need a different tool, and that is the next topic.

## 19.Composite actions
In this lesson you will learn what a composite action is, the action.yml file that defines one, and how a workflow in the same repository uses it.

Since the using-actions topic, you have used actions written by other people, like actions/checkout. A composite action is an action you write yourself. It bundles a group of steps under one name, so any job can run the whole group with a single uses: line.

The pixelpost pipelines all start their logs the same way: print the branch, print the commit, print the runner's Python version. Three echo steps, copied into every job. That is a job for a composite action.

#### The action.yml file
Every action is a folder containing a file named action.yml. For a local action, the folder lives inside your repository, by convention under .github/actions/:
```
name: build-info
description: Print branch, commit, and Python version
runs:
  using: composite
  steps:
    - run: echo "Branch $GITHUB_REF_NAME"
      shell: bash
    - run: echo "Commit $GITHUB_SHA"
      shell: bash
    - run: python3 --version
      shell: bash
```
Three parts to notice:

- runs: using: composite - this is what makes it a composite action, a list of steps rather than a program.
- The steps look like workflow steps, with one extra rule: every run: step must declare shell:. In a workflow the runner picks a default shell; inside an action you must say it.
- Actions can declare inputs: too, a later lesson in this topic covers that.

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
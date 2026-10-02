## Continuous integration
In this lesson you will learn what continuous integration (CI) is and what a CI check does on every push.

Back to the pixelpost team. Their worst Fridays happened because broken code sat unnoticed in the shared codebase for days. By the time someone found it, three other changes were stacked on top.

Continuous integration attacks that delay. The idea: every time anyone pushes code, a machine immediately builds the project and runs the tests. Not on Friday. Now.

#### What a CI run looks like
A typical CI run does three things:
```
Download the exact code that was just pushed.
Install the project's dependencies from scratch, so the run does not depend on whatever is installed on someone's laptop.
Run the checks: tests, a linter, maybe a build.
```

If any step fails, the push is marked with a red cross, and the team knows within minutes. The person who pushed still remembers what they changed, so the fix is quick.

#### Why "integration"
The name comes from integrating, meaning combining, everyone's changes into one shared codebase. Doing it continuously means every push, not once a week. Small merges with instant feedback beat one giant risky merge.

### Continuous delivery and deployment
In this lesson you will learn what the CD part of CI/CD means, and the difference between continuous delivery and continuous deployment.

CI answers one question: is this change safe? CD answers the next one: how does a safe change reach users?

Both CD terms describe automating the path from "tests passed" to "running on the server". They differ in one detail: who presses the final button.

##### Continuous delivery: a human presses the button
With continuous delivery, every change that passes CI is automatically packaged and made ready to release. The actual release to users still needs a human decision. Someone clicks "deploy" when the team is ready.

Banks and hospitals often work this way. The automation does all the mechanical work; a person picks the moment.

##### Continuous deployment: no button at all
With continuous deployment, there is no button. A change that passes every check goes to users automatically, minutes after the push. Many web companies release dozens of times a day like this.

The trade is simple. Continuous deployment gives speed and demands very strong tests. Continuous delivery gives a human checkpoint and costs a little speed.

In this skill you will build both kinds of pipeline with GitHub Actions. In the next lesson: a scenario. Your team debates whether automation is worth the effort, and you weigh in.

## 2.What is github actions.
The five building blocks
In this lesson you will learn the five pieces every GitHub Actions setup is made of: event, workflow, job, step, and runner.

In the last topic, the pixelpost team decided to automate their tests. GitHub Actions is the tool they will use. Before writing anything, you need the vocabulary, because every workflow file you ever read uses these five words.

Event: something that happens in the repository. A push, a pull request being opened, a timer firing. Events are the "when".

Workflow: a YAML file that says "when this event happens, do these things". One repository can have many workflows.

Job: a group of steps inside a workflow that runs on one machine. A workflow can have several jobs, and by default they run at the same time.

Step: one single action inside a job. Run the tests. Install Python. Download the code. Steps run one after another, in order.

Runner: the machine that actually executes a job. GitHub-hosted runners use a fresh virtual machine for each job. Usage limits and billing depend on the repository and plan.

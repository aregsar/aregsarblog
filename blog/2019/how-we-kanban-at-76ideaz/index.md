# How we Kanban at 76ideaz

Mar 21, 2019 by [Areg Sarkissian](https://aregsar.com/about)

[How we Kanban at 76ideaz](https://aregsar.com/blog/2019/how-we-kanban-at-76ideaz)

In this post I will describe our unique Kanban workflow at my web software consulting company, 76ideaz.

Whether working solo or within a cross functional team, the way we use the kanban board is based on the Kanban workflow process that we continue iterating on as we determine the most efficient workflow for our company.

We use Kanbanflow for out collaborative kanban boards at 76ideaz. Its a simple to use kanban board that integrates comments and task lists on each kanban card in addition to a pomodoro timer attached to the card.

## Our kanban board setup

We have only two columns:

+ a `doing` column
+ a `done` column

The done column contains a list of items that are done, where the definition of done, means that the item is deployed to production.

We have only one swim lane which means that entire board is one big swim lane. This means that the `doing` column can only contain a single task that is the highest priority task being worked on.

In kanban terms, this means that we have a work in process (WIP) limit of one.

That's it. A very simple setup. At all times each team or solo developer is working only on a single highest priority item, until that item is done and they start working on the next high priority item.

## Why is our board so spartan

Our kanban board does not have a `todo` items column. We also have dont have a `on deck` or `ready todo` items column.

Consequently so we have no pre planning, no grooming, no estimating (No Estmates) of tasks to be done before actually starting a new task.

We can be in only two states with a task, either we are `doing` the task __right now__ or we are `done` with that task and are `doing` the next task.

Setting up the board this way gives us the following workflow benefits:

+ Ability to focus on the one task at hand:

Since there is a no `todos` column, we don't have to do any planning on many tasks before starting work and then forget what we did once its time to start the task or things may have changed by then.

Instead we do `just in time planning` as we start work on the new `highest priority task` that we add to the `doing` item column.

This allows the team to focus on one thing only and that one thing is the highest priority item that needs to be worked on, working to complete it as fast as possible without distractions.

+ Getting to finish the one thing that we started, all the way:

Our setup allows us to finish what we start completely before starting on the next task. In other words complete one task all the way before moving on to next.

This also means we don't context switch between multiple tasks which eliminates mental overhead and cognitive load that in turn makes us more efficient.

So the workflow starts from just in time requirements planning at the start when a task is added to the `doing` column, to instrumented code deployment at the end when a task is moved to the `done` column.

The task is not done until it's coded, tested, instrumented for monitoring in production and deployed to production.

+ Allows us to show measurable progress:

Since we complete one task before moving on to another we can actually show what we have really completed. 

We get to show completed tasks faster because we dont have multiple tasks in incomplete states.

Furthermore the application is always in the latest working state after each task is completed which means it is deployed to the application in production.

## Summary of the benefits of limiting the board columns and swim lanes

+ we avoid context switching overhead because of no multitasking
+ we avoid having multiple tasks in various incomplete states
+ we avoid the big merge hell because we merge in one task at a time when the task is done and deployed to production and is working in production.
+ we avoid the tracking and fixing defects after big multi-feature release
+ we avoid big rollbacks and can quickly fix any issues and keep rolling forward
+ we avoid a task that is marked as done to switch to another task and then later try to deploy it to production and discover we have a problem an the task was actually not completely done.

## Our kanban workflow

Each task card on our kanban board represents a sub feature which is an endpoint as defined by endpoint driven development where an endpoint represents a vertical slice described here by the Vertical slice architecture.

Our teams are cross functional where all members collaborate on all parts that are required to develop and deploy an `endpoint` sub task.

## Mitigation of blockers

+ Plowing through blockers

We handle blockers focusing all team members on tackling the blocker and expediting any outside resources needed to remove blockers as they emerge. We call this `mobstorming` or `moblitz`.

+ Owning the dependancies

All dependancies become sub task of the `doing` task and will be developed, tested and deployed as part of the task by the team or individual that is working on the task.

An example is when an `endpoint` sub feature task requires a new middleware, data access component or new operating infrastructure for the deployment, as a dependancy.

+ Cuting some slack

Sometimes you cant avoid waiting on another team member in a dedicated cross functional team without shared resourses. This usually happens when a designed is waiting on a programmer or vice versa.

In these scenarions, when waiting on other team members, a member can work on improving the workflow by writing tooling, utilities and documentation or any other means of making the workflow more efficient for subsequent tasks.

Also the wait time can be used by team members to do reseach, experimentation, spikes and prototyping work that helps the team in the long run and helps the team grow in experience.

We think it is better for the company and team members, to have slack time then to have team members jumping to other tasks and switching back or multi tasking.

## Conclusion

Our unique kanban process and board setup is optimized to make us stay focus, avoid multitasking and context switching and deliver truely completed production worthy features faster.

Thanks for reading
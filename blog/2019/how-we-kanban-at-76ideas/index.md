# How we Kanban at 76ideas

In this post I will describe our unique Kanban workflow at my consulting company,  76ideaz.

Whether working solo or within a cross functional team, the way we use the kanban board is based on the Kanban workflow process that we continue iterating on as we determine the most efficient workflow for our team.

We use Kanbanflow for out kanban boards at 76ideaz. Its a simple to use kanban board that integrates comments and task lists on each kanban card in addition to a pomodoro timer attached to the card.

## Our board setup

We have only two columns:

+ a `doing` column
+ a `done` column

The done column contains a list of items that are done, where the definition of done, means that the item is deployed to production.

We have only one swim lane:

+ The entire board is one big swim lane

So the `doing` column can only contain a single task that is the highest priority task being worked on.

This means that we have a work in process (WIP) limit of one.

That's it.

## Why is our board so spartan

Our kanban board does not have a `todos` column so we have no pre planning, no grooming, no estimating (No Estmates) of tasks to be done. We have no `on deck` or `ready todo` column.

Its just binary, either we are working on task or we are done with that task.

Setting up the board this way gives us the following workflow benefits:

+ Focus on the one task at hand:

Since there is a no `todos` column, we don't have to do any planning on many tasks before starting work.

Instead we do `just in time planning` as we start work on the new `highest priority task` that we add to the doing item column.

This allows the team to focus on one thing only and that one thing is the highest priority item that needs to be worked on, working to complete it as fast as possible.

+ Finish the one thing that you started all the way:

Our setup allows us to finish what we start completely before moving on to the next task. In other words Complete one task all the way before moving to next.

So the workflow starts from just in time requirements planning at the start when a task is put in the `doing` column, to instrumented code deployment at the end when a task is moved to to the `done` column.
The task is not done until it's coded, tested, instrumented for monitoring and deployed to production.

+ Show measurable progress:

Since we complete one task before moving on to another we can actually show what we have really completed. Furthermore the application is always in a working state after each task is completed.

## Benefits of limiting the columns and swim lanes

+ we avoid context switching overhead because of no multitasking
+ we avoid having multiple tasks in various incomplete states
+ we avoid the big merge hell
+ we avoid the tracking and fixing defects after big multi-feature release 
+ we avoid big rollbacks and can quickly fix any issues and keep rolling forward
+ we avoid mismarking a task as done becasue we have to switch to another before actually deploying to production.

## the workflow process

Each task card on the kanban board represents a sub feature which is an endpoint as defined by endpoint driven development where an endpoint represents a vertical slice described here by the Vertical slice architecture.

Note: from here on out I will refer to a task or sub feature as an endpoint.

Our teams are cross functional, where all member collaborate on all parts required to build and deploy the endpoint.

## Mitigation of blockers

+ Plow through blockers

We try focusing all team members on tackling the blocker and expediting any outside resources needed to remove blockers as they emerge.

+ Own the dependancies

All dependancies become part of the task. Doing the minimum amount of work needed to get the endpoint completed.

For instance an endpoint requiring a new middleware, generic data access component or new operating infrastructure for the deployment. 

+ Cut some slack

When waiting on other team members, a member can work on improving the workflow by writing tooling, utilities and documentation and any other means of making the work flow more efficient for subsequent tasks.

Also wait time can be used by team members to do reseach, experimentation, spikes and prototyping work that helps the team in the long run and helps them grow.
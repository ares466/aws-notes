# SWF

The `Simple Workflow Service` (SWF) is an AWS service that allows users to build an automated workflow that is coordinated over distributed components.

SWF is not serverless - it uses EC2 instances and servers.

> [Exam Tip]
>
> SWF is the predecessor to Step Functions. It is considered a legacy service that should not be used unless Step Functions does not meet the requirements.
> Choose SWF when...
> - `AWS Flow Framework`
> - Require external Signals to intervene in the process
> - Launch child flows
> - Bespoke/complex decisions
> - `Mechanical Turk`

For everything else (including 99.9% of use cases), choose Step Functions.

SWF workflows contain `activity tasks` (the logical representation of a worker) and `activity workers` (the component that performs the task). SWF worklows can also have `deciders` to implement branches within a workflow.

SWF has a `1 year maximum duration`.

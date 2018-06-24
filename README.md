# TrivialCi

The CI without features.

Just a concept preview for now.

## Vision

After using Jenkins for a while, I propose that:

- It's easier to create a new CI from scratch than configure Jenkins.
- Trying to do everything the opposite way from Jenkins is a good starting point
- CI should have almost no built in features. It should listen to some events and launch scripts

## Topology

- Single Hub that runs scripts as responses to events
- Multiple Workers that runs scripts as instructed by Hub
- Workers can send events to Hub (work_required)
- Hub and Worker can be on same machine. I.e. you don't need "cloud", containers or whatnot, and you can debug the flow on local machine.








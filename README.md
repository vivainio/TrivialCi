# TrivialCi

The CI without features.

Just a concept preview for now.

## Vision

After using Jenkins for a while, I propose that:

- It's easier to create a new CI from scratch than configure Jenkins. Groovy must die in fire.
- Trying to do everything the opposite way from Jenkins is a good starting point. E.g. no plugins, no JVM, no pipeline config.
- CI should have almost no built in features. It should listen to some events and launch scripts
- It should be blazing fast. I.e. slaves and Hub maintain fully "git-fetched" repo in global area and do a "git worktree add" for working areas. This rules out some PaaS systems like Lambda. Own disk is king.

## Topology

- Single Hub that runs scripts as responses to events
- Multiple Workers that runs scripts as instructed by Hub
- Workers can send events to Hub (work_required)
- Hub and Worker can be on same machine. I.e. you don't need "cloud", containers or whatnot, and you can debug the flow on local machine. 
- Scripts get payloads with the data they require (branch info or whatever) either as base64 encoded json in argv or clean json from stdin. So yeah, you probably want to use language that can deal with this like Python to write these scripts.
- There is no persistent state outside Hub and the artifacts from the jobs uploaded by Workers (in S3 or wherever). TBD where Hub state should be persisted (it's mostly append only event log, so json in file system should be ok?)
- Hub state should not be valuable. The history of old jobs should be fully discoverable from static artifacts.
- Workers don't flush console logs etc to Hub (to minimize the load on Hub node). If you want to tail it, Worker can expose it with a local web server that you can navigate to from small web UI on Hub 

## Example flow:

- Hub is booted up with the information about config (mostly repo it needs to watch, and the branch that contains the code for the Hub  itself)
- `branch_update` event sent by GitWathcer daemon to Hub, pipeline/branch_update.py gets run on Hub
- branch_update.py:
    - checks the branch name and notices it's a branch that needs to checked by CI (e.g. it doesn't start with tmp/)
    - notes that there are no Workers. It does the needfull to start booting up a worker (either locally or in cloud) with 
      the information about address & port of the Hub
- Worker comes online and sends `work_required` event to Hub
- Hub runs pipeline/work_required.py. This
  - Checks some local data on what work is available for that kind of Worker (e.g. builder, integrationtester)
  - Hub sends the work package over in `start_work` event to Worker.
- Worker runs pipeline/start_work.py that:
  - Does a git fetch on the global repo it already has mostly up to date
  - Does "git worktree add /my/workpace/1" or wherever for the right sha
  - Starts the build
  - Uploads the artifacts
  - Sends `work_status` updates to Hub (e.g. STARTING, BUILDING, FAILED), so that Hub can show the statuses of all the jobs it has dispatched in a small optional UI
  - Sends `work_done` event to the Hub
  - Sends new `work_required` event to get new work now that it's done!

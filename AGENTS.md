# AGENTS.md

Instructions for AI coding agents working on this NethServer 8 module.

## NethServer skills

General NethServer conventions live in the agent skills of the
[NethServer/agents](https://github.com/NethServer/agents) repository:
module layout, actions and events, backup and restore, UI, Containerfiles,
commits, pull requests and releases. Install them in your agent as that
repository's README explains.

Load the matching skill before you start a task, for example
`nethserver-ns8-module` to change the module code, or `nethserver-admin`
to test it on an NS8 node. Do not copy the skills' rules into this file.

## Module instructions

Read [README.md](README.md) for what this module does, and how to install
and configure it.

Add here only the instructions specific to this module, that the skills
and the README do not cover. For example: the upstream application and
its quirks, the configuration attributes and where they are stored,
external services, how to run the tests, known pitfalls.

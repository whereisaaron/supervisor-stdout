# supervisor-stdout

A simple [supervisord](http://supervisord.org/) event listener to relay
process output to supervisor's stdout.

This is useful in situations where the output will be collected and set to
external logging framework, such as Heroku, Docker, Kubernetes, or AWS ECS/Fargate.

## Installation

Just install via pip or add to your requirements.txt:

    pip install supervisor-stdout

## Installation for Docker

If you're building a Docker container and you find Linux distro has outdated `supervisor` packages,
that the pip `supervisor-stdout` package is very outdated, that `venv` makes `pip` use difficult,
and that `pipx install` doesn't work because `supervisord` can't resolve the handler name,
then you might like to forceably install the latest version from github as follows.

```
RUN pip install git+https://github.com/Supervisor/supervisor.git --break-system-packages && \
    pip install git+https://github.com/coderanger/supervisor-stdout.git --break-system-packages
```

If you find that the 'coderanger/supervisor-stdout' repo hasn't merged this pull request,
then can use this person's fork as follows.

```
RUN pip install git+https://github.com/Supervisor/supervisor.git --break-system-packages && \
    pip install git+git+https://github.com/whereisaaron/supervisor-stdout.git --break-system-packages
```

## Usage

An example `supervisord.conf`:

    [supervisord]
    nodaemon = true
    logfile=/dev/null
    logfile_maxbytes=0

    [program:web]
    command = ...
    stdout_events_enabled = true
    stderr_events_enabled = true
    stdout_logfile = NONE
    stderr_logfile = NONE

    [eventlistener:stdout]
    command = supervisor_stdout
    buffer_size = 100
    events = PROCESS_LOG
    result_handler = supervisor_stdout:event_handler
    stdout_logfile = NONE
    stderr_logfile = NONE

## Advanced Usage

The default `event_handler` prefixes log lines with the process name and channel,
e.g. `apache stderr | <your log line here>`

If you would prefer simpler prefix, or no prefix at all, you can use the alternative
hander names `process_event_handler` and `plain_event_handler`.

`process_event_handler` prefixes logs line with just the process name, e.g. `apache | <your log line here>`.

    [eventlistener:stdout]
    command = supervisor_stdout
    buffer_size = 100
    events = PROCESS_LOG
    result_handler = supervisor_stdout:process_event_handler
    stdout_logfile = NONE
    stderr_logfile = NONE

`plain_event_handler` passed the log line unaltered, without any prefix, e.g. `<your log line here>`.

    [eventlistener:stdout]
    command = supervisor_stdout
    buffer_size = 100
    events = PROCESS_LOG
    result_handler = supervisor_stdout:plain_event_handler
    stdout_logfile = NONE
    stderr_logfile = NONE

## Change Log

Forked from @coderanger's excellent [`supervisor-stdout`](https://github.com/coderanger/supervisor-stdout/) plug-in at the following commit.

https://github.com/coderanger/supervisor-stdout/commit/973ba19967cdaf46d9c1634d1675fc65b9574f6e

Changes:
- Eliminated the blank line emitted after every log line
- Eliminated the block buffering lag that delayed log lines being relayed to supervisor's stdout
- Added alternative handler names that enable relaying log lines unaltered, or with only the process name.

Breaking Changes:
- None, remain compatible with original forked version

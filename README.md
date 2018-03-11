# CADDY-RUN [![Build Status](https://travis-ci.org/lucaslorentz/caddy-run.svg?branch=master)](https://travis-ci.org/lucaslorentz/caddy-run)

## Introduction
This plugin enables caddy to run executables as background processes.

## How it works
For every **run** caddy directive a command is executed in background and killed when caddy stops.

## Caddyfile syntax
The simplest configuration requires only to specify the command the args:
```
run command args
```

Use a block for more control:
```
run {
  command command
  args arg1 arg2 arg3
  dir directory
  env VARIABLEA=VALUEA
  env VARIABLEB=VALUEB
  redirect_stdout file
  redirect_stderr file
  restart_policy policy
  termination_grace_period period
}
```

- **command**: the command or executable name to be executed
- **args**: args provided to the command, separated by whitespace
- **dir**: the working directory the command should be executed in
- **env**: declare environment variable that should be passed to command. This property can be repeated
- **redirect_stdout**: redirect command stdout to a file. Use "stdout" to redirect to caddy stdout
- **redirect_stderr**: redirect command stderr to a file. Use "stderr" to redirect to caddy stderr
- **restart_policy**: define under which conditions the command should be restarted after exit. Valid values:
  - **never**: do not restart the command
  - **on_failure**: restart if exit code is not 0
  - **always**: always restart
- **termination_grace_period**: amount of time to wait for application graceful termination before killing it. Ex: 10s

On windows **termination_grace_period** is ignored and the command is killed immediatelly due to lack of signals support.

## Exponential backoff
To avoid spending too many resources on a crashing application, this plugin makes use of exponential backoff.

That means that when the command exits, it will be restarted with a delay of 10 seconds.
On every successive exit, the delay time doubles, with a max limit of 5 minutes.

If the command runs stable for at least 10 minutes, the restart delay is reset to 10 seconds.

## Examples
AspNet Core application on windows:
```
example.com {
  run {
    env ASPNETCORE_URLS=http://localhost:5000
    command dotnet ./MyApplication.dll
    dir "C:\MyApplicationFolder"
    redirect_stdout stdout
    redirect_stderr stderr
    restart_policy always
  }
  proxy / localhost:5000 {
    transparent
  }
}
```

Php fastcgi on windows:
```
example.com {
  run {
    command ./php-cgi.exe
    args -b 9800
    dir C:/php/
    redirect_stdout stdout
    redirect_stderr stderr
    restart_policy always
  }
  root C:/Site
  fastcgi / localhost:9800 php
}
```

## Building it
Build from caddy repository and import  **caddy-run** plugin on file https://github.com/mholt/caddy/blob/master/caddy/caddymain/run.go :
```
import (
  _ "github.com/lucaslorentz/caddy-run/plugin"
)
```

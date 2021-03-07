---
title : "cli"
description: "Reference documentation for the cli fn"
lead: "The cli fn runs and interacts with an external program over I/O streams."
date: 2021-03-01T01:59:40+00:00
lastmod: 2021-03-01T01:59:40+00:00
draft: false
images: []
menu:
  docs:
    parent: "fns"
weight: 200
---

The cli fn executes an external program and communicates with it over stdin, 
stdout, and stderr. It will publish inputs to the stdin stream and read outputs
from the stdout stream, and it expects each input and output to be expressed on
a single line of text. The cli fn will also read any messages printed to the
program's stderr stream and publish it to the runner's stdout.

The cli fn supports two different types of external programs: long-running 
programs that continue running between invocations and scripts that are started
with each input. The fn assumes the external program is long-running by default,
and it will restart the program if it crashes. Long-running programs are useful
when maintaining resources such as connections between invocations.

The cli fn guarantees that an external program will only receive one input to
process at a time, so the programs do not need to handle requests concurrently
internally and may be written in a synchronous style. This approach provides
process-level isolation for requests. Coupled with the fn restarting fns that
exit unsuccessfully, developers may take a "let it crash" approach when
developing business functions.

## fnrunner Key
`fnrun.fn/cli`

## Configuration
The cli fn may be configured with a string or object value. If configured with
a string value, it is assumed that the value is the command value and all other
values will be default.

The following table describes the keys and expected values when configuring with
an object.

| Key       | Type        | Default | Description                                                                                          |
|-----------|-------------|---------|------------------------------------------------------------------------------------------------------|
| `command` | String      | `""`    | The command to run the external program (e.g., `node ./myfn.js`)                                     |
| `env`     | String List | `[]`    | A list of env vars in the form `VAR=value`                                                           |
| `script`  | Boolean     | `false` | A value indicating whether an instance of the external program must be started with every invocation |


## Examples
The following example demonstrates executing a long-running function that is
provided the input of an http body as input.

```yaml
source: 
  fnrun.source/http:
    treatOutputAsBody: true
middleware:
  - fnrun.middleware/key: body
fn: 
  fnrun.fn/cli: node ./fn.js
```

The following example demonstrates the same scenario as the previous with the
exception that the external program is a script.

```yaml
source: 
  fnrun.source/http:
    treatOutputAsBody: true
middleware:
  - fnrun.middleware/key: body
fn: 
  fnrun.fn/cli: 
    command: ./fn.sh
    script: true
```

In this final example, the external program will have an env var `MY_VAR` set to
value `1` when it is executed.

```yaml
source: 
  fnrun.source/http:
    treatOutputAsBody: true
middleware:
  - fnrun.middleware/key: body
fn: 
  fnrun.fn/cli: 
    command: node ./fn.js
    env:
      - MY_VAR=1
```
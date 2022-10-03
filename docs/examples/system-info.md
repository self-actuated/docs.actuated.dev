# Example: Get system information about your microVM

This sample reveals system information about your runner.

Certified for:

- [x] `x86_64`
- [x] `arm64` including Raspberry Pi 4

## Try out the action on your agent

Create a specs.sh file:

```bash
#!/bin/bash

hostname

whoami

echo Information on main disk
df -h /

echo Memory info
free -h

echo Total CPUs:
echo CPUs: nproc

echo CPU Model
cat /proc/cpuinfo |grep "model name"

echo Kernel and OS info
uname -a

cat /etc/os-release

echo PATH defined as:
echo $PATH
```

Create a new file at: `.github/workspaces/build.yml` and commit it to the repository.

```yaml
name: CI

on:
  pull_request:
    branches:
      - '*'
  push:
    branches:
      - master

jobs:
  specs:
    name: specs
    runs-on: actuated
    steps:
      - uses: actions/checkout@v1
      - name: Check specs
        run: |
          ./specs.sh
```

See also: [actuated-samples/specs](https://github.com/actuated-samples/specs/)

Note how the hostname changes every time the job is run.
# Run a test build

Now that you've [registered your GitHub organisation](/register), [created a server](/provision-server), and [configured the agent](/install-agent), you're ready for a test build.

We recommend you run the following build without changes to confirm that everything is working as expected. After that, you can modify an existing build and start using actuated for your team.

The below steps should take less than 10 minutes.

## Create a repository and workflow

This build will show you the specs, OS and Kernel name reported by the MicroVM.

1. Create a test repository and a GitHub Action

    Create `./.github/workflows/ci.yaml`:

    ```yaml
    name: CI

    on:
        pull_request:
            branches:
            - '*'
        push:
            branches:
            - master
            - main
        workflow_dispatch:

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

    Note that the `runs-on:` field says `actuated` and not `ubuntu-latest`. This is how the actuated control plane knows to send this job to your agent.

    Then add `specs.sh` to the root of the repository:

    ```bash
    #!/bin/bash

    echo Information on main disk
    df -h /

    echo Memory info
    free -h

    echo Total CPUs:
    echo CPUs: $(nproc)

    echo CPU Model
    cat /proc/cpuinfo |grep "model name"

    echo Kernel and OS info
    uname -a

    if ! [ -e /dev/kvm ]; then
        echo "/dev/kvm does not exist"
    else
        echo "/dev/kvm exists"
    fi

    echo OS
    cat /etc/os-release

    echo Egress IP:
    curl -s -L -S https://checkip.amazonaws.com
    ```

2. Hit commit, and watch the VM boot up.

    You'll be able to see the runners registered for your organisation on the [Actuated Dashboard](https://dashboard.actuated.dev) along with the build queue and stats for the current day's builds.

3. If you're curious

    You can view the logs of the agent by logging into one of the Actuated Servers with SSH and running the following commands:

    ```bash
    sudo journalctl -u actuated -f -o cat

    # Just today's logs:
    sudo journalctl -u actuated --since today -o cat
    ```

    And each VM writes the logs from its console and the GitHub Actions Runner to `/var/log/actuated/`.

    ```bash
    sudo cat /var/log/actuated/*
    ```

Do you have any questions or comments? Feel free to reach out to us over Slack in the public channel for support.

## Enable actuated for an existing repository

To add actuated to an existing repository, simply edit the workflow YAML file and change `runs-on:` to `runs-on: actuated`.

If you want to go back to a hosted runner, edit the field back to `runs-on: ubuntu-latest` or whatever you used prior to that.

## Recommended: Enable a Docker Hub mirror

We provide an add-on for setting up a cache/mirror of the Docker Hub. If you do not enable this, and use the Docker Hub in your builds, then you may run into the rate limits imposed by Docker Hub for anonymous users.

See also: [Set up a registry mirror](/examples/registry-mirror)

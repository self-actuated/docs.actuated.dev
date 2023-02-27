# Add your first agent to actuated

actuated is split into three parts:

1. An Actuated Agent (agent) that you run on your own machines or VMs (server), which can launch a VM with a single-use GitHub Actions runner.
2. A VM image launched by the agent, with all the preinstalled software found on a hosted GitHub Actions runner.
3. Our own control plane that talks to GitHub on your behalf, and schedules builds across your fleet of agents.

We look after 2 and 3 which means you just have to set up one or more agent to get started.

!!! info "Have you registered your organisation yet?"
    Before you can add an agent, you or your GitHub organisation admin will need to install the: [Actuated GitHub App](register).

### Pick your Actuated Servers

Pick your Actuated Servers carefully using our guide: [Pick a host for actuated](/provision-server)

## Review the End User License Agreement (EULA)

Make sure you've read the [Actuated EULA](https://github.com/self-actuated/actuated/blob/master/EULA.md) before registering your organisation with the actuated GitHub App, or starting the agent binary on one of your hosts.

## Install the Actuated Agent

1. Download the Actuated Agent and installation script to the server

    > Setting up an ARM64 agent? Wherever you see `agent` in a command, change it to: `agent-arm64`. So instead of `agent keygen` you'd run `agent-aarch64 keygen`.

    Install [crane](https://github.com/google/go-containerregistry/releases):

    ```bash
    curl -sLS https://get.arkade.dev | sudo sh
    arkade get crane
    sudo mv $HOME/.arkade/bin/crane /usr/local/bin/
    ```

    Download the latest agent and install the binary to `/usr/local/bin/`:

    ```bash
    rm -rf agent
    mkdir -p agent
    crane export ghcr.io/openfaasltd/actuated-agent:latest | tar -xvf - -C ./agent

    sudo mv ./agent/agent* /usr/local/bin/
    ```

    Run the setup.sh script which will install all the required dependencies like containerd, CNI and Firecracker.

    ```bash
    cd agent
    sudo ./install.sh
    ```

    Create a file to store your license. If you don't have it handy, check your email for your receipt.

    ```bash
    mkdir -p ~/.actuated

    # Paste the contents, hit enter, then Control + D
    # Or edit the file with nano/vim
    cat > $HOME/.actuated/LICENSE
    ```

2. Generate your enrollment file

    You can generate an enrollment file at `$HOME/.actuated/agent.yaml` by running:

    ```bash
    agent enroll --url https://server1.example.com
    ```

    The enrollment file contains:

    * The hostname of the server
    * The public key of the agent which we use to encrypt tokens sent to the agent to bootstrap runners to GitHub Actions
    * A unique API token encrypted with our public key, which is used by the control plane to authenticate each message sent to the agent

4. Expose the agent's endpoint using HTTPS

    The actuated control plane will only communicate with a HTTPS endpoint to ensure properly encryption is in place. An API token is used in addition with the TLS connection for all requests.

    In addition, any bootstrap tokens sent to the agent are further encrypted with the agent's public key.

    For hosts with public IPs, you will need to use the built-in TLS provisioning with Let's Encrypt. For hosts behind a firewall, NAT or in a private datacenter, you can use inlets to create a secure tunnel to the agent.

    We're considering other models for after the pilot, for instance GitHub's own API has the runner make an outbound connection and uses long-polling.

    See also: [expose the agent with HTTPS](/expose-agent/)

5. Start the agent

    For an Intel/AMD Actuated Agent, create a `start.sh` file:

    ```bash
    #!/bin/bash

    echo Running Agent from: ./agent
    DOMAIN=agent1.example.com

    sudo -E agent up \
        --image-ref=ghcr.io/openfaasltd/actuated-ubuntu20.04:x86-64-latest \
        --kernel-ref=ghcr.io/openfaasltd/actuated-kernel-5.10.77:x86-64-latest \
        --letsencrypt-domain $DOMAIN \
        --letsencrypt-email webmaster@$DOMAIN
    ```

    For an Actuated Agent behind an [inlets tunnel](https://inlets.dev):

    ```bash
    #!/bin/bash

    echo Running Agent from: ./agent
    sudo -E agent up \
        --image-ref=ghcr.io/openfaasltd/actuated-ubuntu20.04:aarch64-latest \
        --kernel-ref=ghcr.io/openfaasltd/actuated-kernel-5.10.77:aarch64-latest \
        --listen-addr 127.0.0.1:
    ```

    For ARM64 Actuated Servers, change the prefix of the image tags from `x86-64-` to `aarch64-`

    You can also run the Actuated Agent software as a systemd unit file for automatic restarts and to start upon boot-up.

    An example of a systemd file you can use:

    ```bash
    cat <<EOF > actuated.service
    [Unit]
    Description=Actuated Agent
    [Service]
    User=root
    Type=simple
    ExecStart=/root/start.sh
    Restart=always
    RestartSec=5s

    [Install]
    WantedBy=multi-user.target
    EOF
    ```
    Double-check the `ExecStart` directive to make sure it includes the path to your `start.sh` file.
    
    The usual place to save the unit file is: `/etc/systemd/system/actuated.service`:

    ```bash
    sudo cp actuated.service /etc/systemd/system/
    ```

    After saving your service file, you can start the service for the first time.

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable --now actuated
    ```

    Verify that it is running:

    ```bash
    systemctl status actuated
    ```

6. Send us your agent's connection info

    Share the `$HOME/.actuated/agent.yaml` file with us so we can add your agent to the actuated control plane.

    We'll let you know once we've added your agent to actuated and then it's over to you to start running your builds.

## Next steps

You can now start your first build and see it run on your actuated agent.

[Start a build on your agent](/test-build)

See also: [Troubleshooting your agent](/troubleshooting)

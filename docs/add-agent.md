# Add your first agent to actuated

actuated is split into three parts:

1. An agent that you run on your own machines or VMs, which can launch a VM with a single-use GitHub Actions runner
2. A VM image launched by the agent, with all the preinstalled software found on a hosted GitHub Actions runner
3. Our own control plane that talks to GitHub on your behalf, and schedules builds across your fleet of agents.

We look after 2 and 3 which means you just have to set up one or more agent to get started.

!!! info "Have you registered your organisation yet?"
    Before you can add an agent, you or your GitHub organisation admin will need to install the: [Actuated GitHub App](register.md).

## Decide where to run your agent

There are three places you can run an agent:

1. Bare-metal on-premises (cheap, convenient, high performance)

    Running bare-metal on-premises is a cost-effective and convenient way to re-use existing hardware investment.

    The machine could be racked in your server room, under your desk, or in a co-lo somewhere.

    Make sure you segment or isolate the agent into its own subnet, VLAN, DMZ, or VPC so that it cannot access the rest of your network. If you are thinking of running an actuated runner at home, we can share some iptables rules that worked well for our own testing.

    For on-premises ARM64 builds, we recommend the Mac Mini M1 (2020) with 16GB RAM and 512GB storage with Asahi Linux. The Raspberry Pi 4 also works, and in one instance [was much faster than using emulation with a Hosted GitHub Runner](https://twitter.com/alexellisuk/status/1583092051398524928?s=20&t=2SelTpdc5idJLmayIu3Djw).

2. Bare-metal on the cloud (higher cost, convenient, high performance)

    Bare-metal doesn't need to mean on-premises. You can deploy machines by API, pay-as-you-go and get the highest performance available.
    
    Bear in mind that whilst the cost of bare-metal is higher than VMs, you will be able to pack more builds into them and get better throughput since actuated can schedule builds much more efficiently than GitHub's self-hosted runner.

    There are at least a dozen options here: Equinix Metal, AWS, Cherry Servers, Alibaba Cloud, OVHcloud, fasthosts, Scaleway and Vultr, [see a list here](https://github.com/alexellis/awesome-baremetal#bare-metal-cloud)
    
    For x86_64 builds we recommend using [Equinix Metal](https://deploy.equinix.com/) for the best price / performance ratio. They also have discounts for reserved instances on contract. The smallest instances available are the c3.small.x86 and c2.small.x86.

    Whilst we don't yet have experience with OVHcloud, they have [a wide range bare-metal servers available](https://www.ovhcloud.com/en-gb/bare-metal/rise/) at a low cost. These tend to be paid for per month.

    For ARM64 builds the cheapest option is to use the [a1.metal](https://aws.amazon.com/ec2/instance-types/a1/) instance on AWS. For a step up on specs, take a look at the c3.large.arm64 from [Equinix Metal](https://metal.equinix.com/).

3. Cloud Virtual Machines (VMs) that support nested virtualization (lowest cost, convenient, mid-level performance)

    This option may not have the raw speed and throughput of a dedicated, bare-metal host, but keeps costs low and is convenient for getting started.

    We know of at least three providers which have options for nested virtualisation: [DigitalOcean](https://m.do.co/c/8d4e75e9886f) [Google Compute Platform (GCP)](https://cloud.google.com/compute) (new customers get 300 USD free credits from GCP) support nested virtualisation on their Virtual Machines (VMs), and [Azure](https://azure.com/).

    We have tested ARM64 VMs on Oracle OCI, Azure and GCP and found that they do not currently allow for virtualisation. So whilst these clouds may be an option for x86, for ARM64, you'll need access to bare-metal.

The recommended Operating System for an Actuated Agent is: Ubuntu Server 22.04 or Ubuntu Server 20.04.

## Review the End User License Agreement (EULA)

Make sure you've read the [Actuated EULA](https://github.com/self-actuated/actuated/blob/master/EULA.md) before registering your organisation with the actuated GitHub App, or starting the agent binary on one of your hosts.

## Set up your first agent

1. Download the agent and installation script

    Once you've decided where to set up your first agent, you'll need to download the installation package from a container registry

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

    Create a file to store your license. If you don't have it handy, go to [gumroad.com](https://gumroad.com), click "Library" then click "View content"

    ```bash
    mkdir -p ~/.actuated

    # Paste the contents, hit enter, then Control + D
    # Or edit the file with nano/vim
    cat > $HOME/.actuated/LICENSE
    ```

2. Generate an RSA keypair

    ```bash
    cd ~/.actuated/
    agent keygen
    ```

    The RSA keypair is only used to encrypt messages and cannot. RSA keys are sometimes used with SSH sessions, however actuated does not use any form of SSH at this time.
    
    This will write: `key_rsa` and `key_rsa.pub` to the current working folder.

3. Install the agent's authentication token.

    Create an API token for us to present when we send jobs to your Actuated Agent:
    
    ```bash
    openssl rand -base64 32 > ~/.actuated/TOKEN
    ```

    Encrypt the token with our public key and email `.actuated/TOKEN.ENC` to us, or share it with us on Slack:

    ```bash
    cat <<EOF > actuated.pem
    -----BEGIN RSA PUBLIC KEY-----
    MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAo9EC7IrP8zTE9jm8agPa
    m0D/sFfmAlchhskLZksO4ZYzDHK9fuQ9oEhPYVkgrU5TifbL5UchdsSn//ELSy2Q
    TPRQoXVMdzPgLCrn15U+Xr7KpV3iNBV1go+ZzNE/ymdyS2kCCjxYiBLVuymn20hA
    ZzqkHSyOeM6IrG+A462KfmN0vqIpubpMkoK/wSkSSDjN0SoMWc9gaAqEFEHkSt9+
    t65fIdzG0sKSEMb613WG+K/A/WBrcdqGHWoMG2h2CpK12tNobZEt3yCL0WVgkAKU
    VwaHniNYHn5niJHH/DgvXMWDECoKA1ZJyMdWC3MuIlyfWVzT5N7a/HPTzyzlrdCl
    bwIDAQAB
    -----END RSA PUBLIC KEY-----
    EOF

    agent encrypt --key ./actuated.pem \
        --in $HOME/.actuated/TOKEN \
        --out $HOME/.actuated/TOKEN.enc
    ```

    Post-pilot, we will provide a more automated way to exchange this token.

4. Add HTTPS for the agent's endpoint

    The actuated control plane will only communicate with a HTTPS endpoint to ensure properly encryption is in place. An API token is used in addition with the TLS connection for all requests.

    In addition, any bootstrap tokens sent to the agent are further encrypted with the agent's public key.

    For hosts with public IPs, you will need to use the built-in TLS provisioning with Let's Encrypt. For hosts behind a firewall, NAT or in a private datacenter, you can use inlets to create a secure tunnel to the agent.

    We're considering other models for after the pilot, for instance GitHub's own API has the runner make an outbound connection and uses long-polling.

    See also: [expose the agent with HTTPS](expose-agent.md)

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

    For ARM64 Actuated Agents, change the prefix of the image tags from `x86-64-` to `aarch64-`

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

    Share the following with us over Slack or email, these details are confidential.

    ```bash
    # Your agent's public key for encrypting messages
    cat ~/.actuated/key_rsa.pub

    # Your agent's encrypted API token
    cat ~/.actuated/TOKEN.enc

    # Your agent's endpoint
    echo "https://agent1.example.com
    ```

    We'll let you know once we've added your agent to actuated and then it's over to you to start running your builds.

## Next steps

You can now start your first build and see it run on your actuated agent.

[Start a build on your agent](test-build.md)

See also: [Troubleshooting your agent](troubleshooting.md)

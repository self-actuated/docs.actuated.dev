# Register your GitHub Organisation

Actuated is a managed service, where you plug in your own servers, and we do everything else needed to create and manage self-hosted runners for your GitHub Actions.

Plans are paid monthly, without any minimum commitment.

## What you'll need

* A GitHub organisation
* One or more public or private repositories hosted in the organisation
* Administrative access to install the actuated GitHub App
* A company credit-card to pay for your subscription
* One or more bare-metal servers (we'll recommend the best fit for you during onboarding)

## We'll guide you through the process

![Onboarding steps](/images/onboarding-steps.png)

> We'll walk you through the onboarding process and answer all your questions, so that you can be up and running straight away.

We've now run over 245,000 VMs for commercial teams, and there's very little for you to do to get started, in most cases, we've seen a 2-3x speed up for `x86_64` builds by switching one line in a workflow: `runs-on: actuated`. For `Arm` builds, native hardware makes a night and day difference.

## Book a call with us

Some engineers hate talking to sales people. You're not alone, and this is not a sales call, or with sales people. Our pricing is public, and paid month to month by corporate card.

The purpose of a call is to understand your goals, and help you pick the best server hardware, hosting company, and subscription plan for your usage.

Before the call, generate a usage report with our open-source [actions-usage tool](https://github.com/self-actuated/actions-usage) for at least 7 days, and either send it over via email or share it with us on the call. It'll help us make a better recommendation.

[Talk to us](https://forms.gle/8XmpTTWXbZwWkfqT6)

## Install the GitHub App

An administrator from your GitHub organisation will need to install the actuated GitHub App. [GitHub Apps](https://docs.github.com/en/apps/creating-github-apps/about-creating-github-apps/about-creating-github-apps) provide fine-grained access controls for third-parties integrating with GitHub.

Learn more in the [FAQ](faq.md). 

!!! info "End User License Agreement (EULA)"
    Make sure you've read the [Actuated EULA](https://github.com/self-actuated/actuated/blob/master/EULA.md) before registering your organisation with the actuated GitHub App.

1. Click on the [Actuated GitHub App](https://github.com/apps/self-actuated)
2. Click **Install app**
3. Select the organisation you want to install the Actuated app to
4. Install the app on all repositories or select repositories

    ![Install GitHub app](/images/install_github_app.png)

5. Once installed you will will see the permissions and other configuration options for the Actuated GitHub App on your selected account. If you have multiple organisations, you will need to authorise each one.

To remove or update the Actuated GitHub app, navigate to "Settings" for your organisation and click on "GitHub Apps" in the left sidebar. Select the Actuated from the list and click "Configure".

## Next steps

Now that you've installed the GitHub App, and picked a subscription plan:

* [Provision a server](/provision-server)

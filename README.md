# Amplify-docusaurus

A sample repo that shows how to setup builds with github actions to deploy
docusaurus to amplify. Additionally, there's an argos setup if you want
screenshot diff tracking.

## Why?

Amplify has incredibly slow build times, low visibility into what is happening
during a build and limited options for when a PR preview is deployed. Other
providers, like Vercel, charge quite a premium for it. However, Github Actions,
can be quite fast, integrated with PRs, and has a deployments section, while
also not being much of an additional cost.

## Does this have to be docusaurus?

No, actually almost any static site should work. I just haven't tried it with
anything else.

## Is this Amplify Gen 1 or Gen 2?

This is amplify Gen 1. It utilizes
[manual builds](https://docs.aws.amazon.com/amplify/latest/userguide/manual-deploys.html).
It may work with Amplify Gen 2 as well, haven't tested it.

## Shouldn't this just be using S3 buckets instead of Amplify?

Probably. I still like some of Amplifies features that S3 buckets don't have.
Feel free to add an alternative version that doesn't use Amplify at all.

## Who would you recommend this for?

Anyone who's already a Github Enterprise user, using Docusuarus, for a team of
less than 20 people. This solution isn't super polished, and it will require
some maintenance to get just right for your team. But it's open source, so feel
free to just take what works for you.

## How does this work?

High level:

1. [When a PR is opened, or edited](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#pull_request),
   it triggers the `AmplifyPreview.yml` workflow, which does the following:

2. [Setup AWS Access](https://github.com/aws-actions/configure-aws-credentials),
   so the runner has permissions to use AWS

3. [Create an Amplify Branch ](https://docs.aws.amazon.com/cli/latest/reference/amplify/create-branch.html)to
   deploy the preview on.

4. [Sets up Node](https://github.com/actions/setup-node), for building
   Docusaurus

5. Runs the preview script, which does:

6. Informs Github of the status of the build using
   [Deployments Rest API](https://docs.github.com/en/rest/deployments/deployments?apiVersion=2022-11-28).

7. Actually builds and bundles Docusaurus

8. [Creates the amplify Deployment](https://docs.aws.amazon.com/cli/latest/reference/amplify/create-deployment.html)
   and then
   [starts the deployment](https://docs.aws.amazon.com/cli/latest/reference/amplify/start-deployment.html).

9. After the PR is closed, the
   [amplify branch is deleted](https://docs.aws.amazon.com/cli/latest/reference/amplify/start-deployment.html).

### Setup required

1. Make an amplify App, Gen 1, with no additional settings. Make sure automatic
   previews are turned off

2. Make an Amplify Admin Role that github actions has access to and then change
   the `configure-aws-credentials` workflow step to include this role.

3. Setup your secrets:

- `DEPLOY_GH_PAT`: Make a github personal access token with read/write access to
  the repository, this is so you can trigger an additional workflow after
  Deployment. See
  [Triggering a workflow from a workflow](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/triggering-a-workflow#triggering-a-workflow-from-a-workflow)
- `AMPLIFY_APP_ID`: This doesn't really have to be a secret, but it's nice to
  not repeat it. This is where previews should be built.

### Optional - Setup Argos

Argos is a very reasonably priced, very well supported screenshot diffing tool.
I'll leave the actual writing of the test up to you, but I'll include a sample
workflow file, that will run after deployment.

Here is [their setup](https://argos-ci.com/docs/quickstart/playwright)

## Prior work

I found a few blog posts that were essential to putting everything together. If
you're struggling with understanding how to extend this code, I recommend
reading these.

The script is based of Invariance.dev
[Continuous AWS Amplify deployment](https://invariance.dev/2021-03-03-continuous-aws-amplyfy-deployment.html)

yinlinchen -
[Amplify Preview actions ](https://github.com/yinlinchen/amplify-preview-actions)

Sander Knape's
[Deploy your pull requests with GitHub Actions and GitHub Deployments](https://sanderknape.com/2020/05/deploy-pull-requests-github-actions-deployments/)

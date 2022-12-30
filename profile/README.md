# Compada Engineering

Welcome to Compada Engineering! If you stumbled upon this, and you're not really sure what we're about, please head over to <https://www.compada.io>; send us your resume if you believe in our mission. If you're new to the organization, we're looking forward to working with you!

Check out <https://github.com/orgs/compada/discussions> for any new announcements and to read up on our [ADRs](https://docs.aws.amazon.com/prescriptive-guidance/latest/architectural-decision-records/adr-process.html). Hopefully this living document can help you get started with our stack, and serve as a North Star if you're ever lost.

## Architecture

> The battle against complexity in web development is a constant tug of war. We give a little to get something new, we take it back to make it simpler.
> Progress is good. Complexity is a bridge. Simplicity is the destination.

Funnily enough, this is a [quote from DHH](https://world.hey.com/dhh/introducing-propshaft-ee60f4f6). For those unfamiliar, DHH is the founder of Ruby's Rails framework, and is a huge proponent of the "mighty monolith." While we disagree with his ideal architecture, we very much subscribe to keeping things simple. In a similar vein, knowing when to buy (or find) something vs build it ourselves is an important part of our jobs. There's no need to be continually reinventing the wheel.

We find microservices to be an ideal starting point. It may seem overkill in the beginning of a project, but it forces you into good habits. Their benefits include:

- Improved scalability
- Better fault isolation for more resilient applications
- Programming language and technology agnostic
- Improved Continuous Integration & Continuous Deployments
- Better data security and compliance
- Faster time to market and "future-proofing"
- Greater business agility and support for devops
- Support for two-pizza development teams

With microservices adopted, and deployed... how does our frontend communicate with our backend? We kill 2 birds with 1 stone on this front with a Federated GraphQL Gateway (aka [supergraph](https://www.apollographql.com/blog/announcement/backend/the-supergraph-a-new-way-to-think-about-graphql/)). This provides a unified graphQL schema as well as intelligent dispatching of requests to the appropriate service (aka subgraph) with no code to manage!

Frontends (web, iOS, android) can be developed and deployed individually, and all connect to the same API via a singular graphQL schema.

## Development Environment

Containerization changed everything. Long ago are the days of needing a version manager for every language you're tinkering with, and trying to remember the syntax of each one. [asdf](https://asdf-vm.com/) solves part of the problem, but [docker FROM](https://docs.docker.com/engine/reference/builder/#from) renders most version tooling obsolete. Pick a language and version (and even platform); now it's not only checked into source code, but you have parity in every environment.

But there's so much more... the Dockerfile allows you to configure a “box” with as much or as little as you desire - and it's reproducible! 🪦 Chef and Ansible. You can easily look at a Dockerfile, and with a basic understanding of how a shell works, know what you can expect from the container it will spin up.

All that is to say that setting up your local environment is as easy as setting up [docker](https://docs.docker.com/desktop/)...

Compose files are included with tooling to run language/framework commands:

```sh
# pro-tip
alias dr='docker compose run --rm '

dr npm install nodemon --save-dev
dr npx graphile-migrate commit
dr node - # opens node CLI

dr go get -d golang.org/x/net
dr go mod tidy

dr bundle install
dr bundle exec rails db:setup
```

Spinning up a local server typically consists of:

```sh
cp .env.sample .env
docker compose up -d
```

## Pre-commit Hooks

We don't want to argue about style... there are linters provided for most languages that cover everything from syntax to formatting to deprecations to complexity. Make sure your editor is setup to use those linters, or get yelled at by the hooks before you can commit your changes.

## Continuous Integration

We're avid believers of protecting ourselves from... ourselves. CI enables us to do things like:

- verify our tests are passing
- check our code against linters
- static code analysis
- build various artifacts
- keep our dependencies current

...all without you really having to know much about the repo. Just like the REPL you get into when you're debugging code, CI let's you know when things aren't ready to be merged just yet, and most importantly, how to rectify things!

We make extensive use of GitHub Actions. Reusable workflows are kept in <https://github.com/compada/compada>, and templates for adding Actions to a new repo are available in <https://github.com/compada/.github/workflow-templates>.

## Continuous Deployment

Our main APIs are containerized and orchestrated via Kubernetes. The majority of our infrastructure embraces GitOps, and is available via <https://github.com/compada/cd>. New docker images are built during CI, and kubernetes polls for new image tags via Flux every few minutes.

## Provisioning Resources

<https://github.com/compada/tf>
GitOps. Terraform. Databases. K8s cluster. Auto scaling k8s.

## Observability

So... your PR was merged. Congratulations! An image was built, tagged, and pushed up to the container registry. Kubernetes has presumably detected the new tag, pulled the image, and started scaling up replica sets. Incoming requests are gradually diverted to the new pods. Your changes are taking traffic. Now what?

Go SLO to go fast. Microservices necessitate advanced observability. Distributed tracing.

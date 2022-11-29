## Test case

Hi, I'm Eric Trigo and this is my take on the assessment given.

### Part One

Given the simplicity of the architecture (only a monolithic app) and the need for going cloud native, these were my initial considerations:

- App should be containerized through a Dockerfile, so that we can

  1. Build new images (after our code changes) that we deploy to a container registry.
  2. Deploy our app by pulling our image from the container registry and running it in a container.

- App should include both the ReactJS and .NET Core code, and Dockerfile should account for dependencies for both, building just one image. Another alternative would be having different repos (and of course images) for frontend and backend, but in our case the added complexity didn't seem worth it.

- Cloud native usually means Kubernetes nowadays, but it seems overkill here to provision a full Kubernetes cluster to run a single pod. As an alternative that fits the simplicity of the situation a bit better, I would use a Containers as a service (CaaS) solution, such as Azure Container Apps/Instances or AWS ECS/Fargate, and just run our app as a managed container that can scale automatically based on a provided configuration (as a side note, I bet that under the hood, the different cloud providers run these managed containers in their own Kubernetes clusters anyway).

- Although I advised against using Kubernetes just yet, requirements can change quickly, so our architecture needs to account for flexibility to adapt quickly. Having our main app containerized helps tremendously, so that even if our system grows to be large enough to require a Kubernetes cluster (more applications that we need to deploy, huge increments in traffic...), we should easily be able to install it into a K8s cluster using the image previously uploaded to the container registry.

So, in summary, our setup should consist of the following:

- A Git repository where we can keep our containerized app's code.
- CI/CD pipelines triggered after certain events on our repo (such as a new pull/merge request, new commits in master/main branch, or simply new releases) that will help development teams work as efficiently as possible. I will elaborate a bit in part two.
- A container registry to keep our main app's image (or any other that we need to include in the future).
- A managed service by a cloud provider that pulls the image from our container registry, and runs it as a container.

To demonstrate, I've put together a very basic setup using the following:

- .NET Core 7 backend (basic .NET Core API)
- ReactJS frontend included as part of the same solution (basic Create React App)
- Dockerfile to build both projects and publish the image of our monolithic application (I've used Docker Hub for my tests: https://hub.docker.com/r/erictrigo/dotnetreact)
- Azure Container Instance as CaaS solution, pulling the image from my personal account in Docker Hub and running it as a container here (I'll keep it up for a few days): http://dotnet-react.g9e3fhbfakc6cvbf.westeurope.azurecontainer.io/
- A bit redundant, but also Azure Container App as Caas, just because I wanted to try and see the differences with ACI, very easy to use a secure connection through Ingress: https://dotnet-react-app.ambitiousplant-588cc5f4.westeurope.azurecontainerapps.io/

### Part Two

Unfortunately, I could not get the GitHub Actions code finished on time (I usually work with GitLab CI/CD and wanted to try GitHub's alternative thinking it would be similar, but I overreached), so I'll describe what I was trying to accomplish regarding CI/CD and how I would set it up for the future:

- Local development can be done using Docker.

- Pull/merge requests should have a "build" step, where a new Docker image from our git branch is built and pushed to the container registry, for it to be picked up in a consequent "review" step, where we can deploy it to a testing environment of choice.

- The "build" step should also be included when new commits get to master (which should only happen after a pull/merge request is approved and merged), in order to build a new image from the master/main branch (which will typically become "master" or "latest" in our container registry's repo).

- A "test" step in order to execute unit tests (or any kind of tests, which could mean different "test" steps for each) should be included in most pipelines, even if it is not required to pass in some cases (for example, in order to test quick changes in pull/merge requests, sometimes as a developer you'd want to be able to freeze your changes in a dev environment even if your tests fail).

- For versioning, I'd include a "publish" step that could be triggered manually (but also be automatized if necessary, depending on the adopted release lifecycle), building and generating and image with the tag provided. Another option would be releasing a new version with every new commit to master and simply work with latest; I've never done this and it kind of scares me, but some companies work like that and in such a simple setup, it may make sense.

- Deploying a new version of the app in production should be as easy as changing the image of the container to the desired version, previously generated through a tag in our git repo.

- Other steps could be included in pipelines, such as code analysis (SonarQube is popular) or vulnerability scans (which may make sense to only trigger for tags/releases, for example).

### Things to do

Too many, but I wanted to mention a few:

- Parameterize as encrypted secrets all the sensitive configuration data we may have in our repo. If necessary, implement a managed solution such as Azure Key Vault or AWS Secrets Manager.

- Add custom OpenTelemetry support to our .NET Core backend (it's built-in, but it can be customized).

- Add a chart to the app to support installation through Helm, which would be useful if it needs to run in Kubernetes.

Thanks for reading!

Eric

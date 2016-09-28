# resource-template

[![Code Climate](https://codeclimate.com/github/JeffDeCola/resource-template/badges/gpa.svg)](https://codeclimate.com/github/JeffDeCola/resource-template)
[![Issue Count](https://codeclimate.com/github/JeffDeCola/resource-template/badges/issue_count.svg)](https://codeclimate.com/github/JeffDeCola/resource-template/issues)
[![Go Report Card](https://goreportcard.com/badge/jeffdecola/resource-template)](https://goreportcard.com/report/jeffdecola/resource-template)
[![GoDoc](https://godoc.org/github.com/JeffDeCola/resource-template?status.svg)](https://godoc.org/github.com/JeffDeCola/resource-template)
[![License](http://img.shields.io/:license-mit-blue.svg)](http://jeffdecola.mit-license.org)

`resource-template` _can be used as a template for developing a concourse ci resource type.
It is tested, built and pushed to dockerhub using concourse ci._

## USE EITHER BASH SCRIPT OR GO

This resource type can use either bash script or go.

Change _ci/Dockerfile_ to either ADD _/assets-go_ or _/assets-bash_.

### USING BASH (default)

The 3 bash script files located in _/assets-bash_.

### USING GO

The 3 bash scripts are located in _/assets-go_ that run _main.go_ with the second argument being
_check_, _in_ or _out_ resepctively. Hence only one file _main.go_ need to be
maintained, rather than three seperate files.

## SOURCE CONFIGURATION

These are just placeholders that you can update where your source is.

* `source1`: Just a placeholder.

* `source2`: Just a placeholder.

## BEHAVIOR

### CHECK (a resource version(s))

_CHECK will mimic getting the list of versions from a resource._

#### stdin

```json
{
  "source": {
    "source1": "sourcefoo1",
    "source2": "sourcefoo2"
  },
  "version": {
    "ref": "123 ",
  }
}
```

123 is the current version.

#### stdout

```json
[
  { "ref": "123" },
  { "ref": "3de" },
  { "ref": "456" }
]
```

456 is the latest version that will be used.

The last number 456 will become the current ref version that will be used by IN.

### IN (fetch a resource)

_IN will mimic fetching a resource and placing a file in the working directory._

#### Parameters

* `param1`: Just a placeholder.

* `param2`: Just a placeholder.

#### stdin

```json
{
  "params": {
    "param1": "Hello Clif",
    "param2": "Nice to meet you"
  },
  "source": {
    "source1": "sourcefoo1",
    "source2": "sourcefoo2"
  },
  "version": {
    "ref": "456",
  }
```

#### stdout

```json
{
  "version":{ "ref": "456" },
  "metadata": [
    { "name": "nameofmonkey", "value": "Larry" },
    { "name": "author","value": "Jeff DeCola" }
  ]
}
```

#### file fetched (fetch.json)

The IN will mimic a fetch and place a fake file `fetched.json` file in the working directory:

### OUT (update a resouce)

_OUT will mimic updating a resource._

#### Parameters

* `param1`: Just a placeholder.

* `param2`: Just a placeholder

#### stdin

```json
{
  "params": {
    "param1": "Hello Jeff",
    "param2": "How are you?"
  },
  "source": {
    "source1": "sourcefoo1",
    "source2": "sourcefoo2"
  },
  "version": {
    "ref": ""
  }
}
```

#### stdout

```json
{
  "version":{ "ref": "123" },
  "metadata": [
    { "name": "nameofmonkey","value": "Henry" },
    { "name": "author","value": "Jeff DeCola" }
  ]
}
```

where 123 is the version you wanted to update.

## PIPELINE EXAMPLE USING PUT

```yaml
jobs:
- name: your-job-name
  plan:
  - get: your-repo-names
    trigger: true
    ...
  - put: resource-template
    params: { param1: "hello jeff", param2: "How are you?" }

resource_types:
  ...
- name: jeffs-resource
  type: docker-image
  source:
   repository: jeffdecola/resource-template
   tag: latest

resources:
  ...
- name: resource-template
  type: jeffs-resource
  source:
    source1: foo1
    source1: foo2
```

GET would look similiar.

## TESTED, BUILT & PUSHED TO DOCKERHUB USING CONCOURSE CI

To automate the creation of the `resource-template` docker image, a concourse ci pipeline
will unit test, build and push the docker image to dockerhub.

![IMAGE - resource-template concourse ci piepline - IMAGE](docs/resource-template-pipeline.jpg)

A _ci/.credentials.yml_ file needs to be created for your _slack_url_, _repo_github_token_,
and _dockerhub_password_.

Use fly to upload the the pipeline file _ci/pipline.yml_ to concourse:

```bash
fly -t ci set-pipeline -p resource-template -c ci/pipeline.yml --load-vars-from ci/.credentials.yml
```

## CONCOURSE RESOURCES IN PIPELINE

As seen in the pipeline diagram, the _resource-dump-to-dockerhub_ uses the resource type
[docker-image](https://github.com/concourse/docker-image-resource)
to push a docker image to dockerhub.

`resource-template` also contains a few extra concourse resources:

* A resource (_resource-slack-alert_) uses a [docker image](https://hub.docker.com/r/cfcommunity/slack-notification-resource)
  that will notify slack on your progress.
* A resource (_resource-repo-status_) use a [docker image](https://hub.docker.com/r/dpb587/github-status-resource)
  that will update your git status for that particular commit.

These resources can be easily removed from the pipeline.
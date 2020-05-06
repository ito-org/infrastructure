# ito Infrastructure

![License](https://img.shields.io/github/license/ito-org/infrastructure)

This repository contains [infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_code) to deploy ito artifacts provided by other repositories on different platforms. Multiple/Competing infrastructure solutions are ecouraged (see [CONTRIBUTING.md](./CONTRIBUTING.md)).

## [Heroku Test Environment](./heroku-test/README.md)

<table width="100%">
  <tr>
    <th align="left">Platform</th>
    <th align="left">Artifact</th>
    <th align="left">Deployed to</th>
    <th align="left">Pipeline</th>
    <th align="left">Links</th>
  </tr>
  <tr>
    <td>
      <a title="Heroku" href="https://heroku.com">
        <img src="https://avatars3.githubusercontent.com/u/23211?s=200&v=4" height="40" />
      </a>
    </td>
    <td>
      <a href="https://github.com/ito-org/api-backend">
        api-backend
      </a>
    </td>
    <td>
      <a href="https://dashboard.heroku.com/apps/ito-heroku-test">
        ito-heroku-test app
      </a>
    </td>
    <td>
      <a href="https://github.com/ito-org/infrastructure/actions?query=workflow%3Aheroku-test">
        <img src="https://github.com/ito-org/infrastructure/workflows/heroku-test/badge.svg" />
      </a>
    </td>
    <td>
      <ul>
        <li>
          <a href="https://ito-heroku-test.herokuapp.com:8080/">
            REST endpoint
          </a>
        </li>
        <li>
          <a href="https://dashboard.heroku.com/apps/ito-heroku-test/logs">
            Logs
          </a>
        </li>
      </ul>
    </td>
  </tr>
</table>

## Automating deployments

<a href="https://github.com/ito-org/infrastructure/actions?query=workflow%3Acreate-pull-request">
  <img src="https://github.com/ito-org/infrastructure/workflows/create-pull-request/badge.svg" />
</a>

The `create-pull-request` pipeline of this repository can be triggered externally via [`repository_dispatch`](https://help.github.com/en/actions/reference/events-that-trigger-workflows#external-events-repository_dispatch) events. This trigger mechanism is meant to be integrated as a deploy job into other repository's pipelines. `repository_dispatch` events carry a `client_payload`, which is a json string with custom attributes. The `create-pull-request` pipeline expects the following attributes:

- **label**: A custom label to distinguish pull requests. This is attached to the pull request and also used as part of the pull request's branch name.
- **script**: A shell command, that is supposed to perform some changes to versioned files (e.g. update hardcoded tags/versions). Those changes are committed to the pull request branch.

Here is a GitHub workflow to trigger pull requests from another repository's pipeline on every pushed tag. It requires a secret called `REPO_ACCESS_TOKEN`, which should be a [personal access token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) with `repo:public_repo` scope:

```yaml
name: Deploy
on:
  push:
    tags:
      - *
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Parse tag name
        id: parse_tag_name
        run: echo ::set-output name=tag::$(echo ${GITHUB_REF:10})
      - name: Request deployment
        uses: peter-evans/repository-dispatch@v1
        with:
          repository: ito-org/infrastructure
          token: ${{ secrets.REPO_ACCESS_TOKEN }}
          event-type: create-pull-request
          client-payload: '{"label": "foo-bar", "script": "echo \"${{steps.parse_tag_name.outputs.tag}}\" > DOCKER_TAG_VERSION"}'
```

# Jenkins

After building remote server you will have **Jenkins** installed on `https://YOUR.DOMAIN/jenkins`. It configured in a way that every anonymous user is an administrator. Protection achieved by setting [basic HTTP authentication](../basic-http-auth) for the whole domain using [Nginx](#nginx).

Here's the main view of Jenkins home screen:

![Home screen](images/home-screen.png)

Every CI server can host as much as needed projects. By default, when you'll finish setting it up, Jenkins will contain two project-related jobs: `<PROJECT>_BUILDER` and `<PROJECT>_PR_BUILDER` (`TEST1_BUILDER` and `TEST1_PR_BUILDER` on the screenshot above). Refer to next code snippet for creating new project on existing server:

```shell
./cikit jenkins-job --project=test2 --limit=<SERVER_NAME_FROM_INVENTORY>
```

- What is [SERVER_NAME_FROM_INVENTORY](../ansible/inventory)?

In addition to the project jobs there are available some additional:

- `BACKUP_PROD_DB` - for creating snapshots of your production database (disabled by default and have to be manually configured).
- `SERVER_CLEANER` - periodically running job (every 24 hours by default) for removing all builds (files and databases).
- `DISK_USAGE_TRIGGER` - periodically running job (every 10 minutes by default) for checking available free space on server's hard drive (will trigger `SERVER_CLEANER` if occupied more than 95%).

## Configuring bot

Bot will manage pull request statuses, react on triggering phrases in comments and publish reports. So we need to [set it up](github-bot)!

## Configuring project jobs

Currently **CIKit** works with GitHub only. For making builds of the project you have to host codebase on GitHub and do several steps to configure the job:

- Set the web URL of a project. ![Pull request builder web URL](images/pr-builder-web-url.png)
- Set the repository URL of a project and configure an access for Jenkins (not needed if project is public). ![Pull request builder repository URL](images/pr-builder-repo.png)

Besides, don't ignore advanced configuration of `GitHub Pull Request Builder`. There's you able to edit the list of administrators (GitHub accounts), users allowed to control CI process via comments on Github, triggering phrases, etc.

![Pull request builder repository URL](images/pr-builder-ghprb.png)

Some explanations of the configurations on the image above:

- **Admin list** - GitHub users which can accept/decline participation of other users.
- **Trigger phrase** - finding this text in a comment to pull request will trigger new build.
- **Skip build phrase** - adding this phrase to title or body of pull request will not trigger a build.
- **White list** - list of GitHub users, which are allowed to participate in a project.
- **List of organizations** - the same as white list, but less text. Specify GitHub organization and all its members will be participants.

## Remarks

### Nginx

Nginx - is a global web server. It serves all connections and proxies requests to Jenkins, Solr and Apache. All requests are secured by [basic HTTP authentication](../basic-http-auth) you've configured during installation the CIKit.

## Technical notes

For Jenkins installation was designed the [cikit-jenkins](../../scripts/roles/cikit-jenkins) Ansible role. All related information located there and here is just the list of useful references to it:

- [Jenkins version](../../scripts/roles/cikit-jenkins/vars/main.yml#L5)
- [List of plugins](../../scripts/roles/cikit-jenkins/defaults/main.yml#L29)

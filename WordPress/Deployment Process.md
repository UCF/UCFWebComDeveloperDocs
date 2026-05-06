We currently host our WordPress sites on a platform called [Pantheon](https://pantheon.io). Pantheon is made up of multiple instances, with each instance having multiple environments. We currently have 6 WordPress instances on Pantheon.

There are two primary ways to deploy code to a Pantheon instance. You can push code to the instance via a git repository that is hosted on Pantheon or you can put the instance in SFTP mode, upload/overwrite files in the DEV environment, and then place it back into Git mode and commit the changes. If you'd like to know more about the SFTP mode in Pantheon, you can[ read more about it here](https://docs.pantheon.io/guides/sftp). However, we always keep our Pantheon instances in [Git mode](https://docs.pantheon.io/guides/git), so we'll cover update processes using that method.

## Key Concepts

- UCF's themes and plugins are all individually tracked in public repositories on GitHub
- UCF has multiple WordPress instances hosted on the Pantheon platform
- Each pantheon instance has an associated Git repository that is hosted by Pantheon. Committing changes to this repository is the primary way code is deployed on Pantheon.
- UCF uses a private packagist repository (for PHP composer) to manage themes and plugins within the Pantheon repository.

## Getting Started

Before we can commit any changes to our Pantheon repository, we first need to pull down the repository. Once you're logged into pantheon.io, you'll need to browse to the instance you're intending to work on, click on the "Connection Info" button at the top of the environment panel, and copy the git clone command. Detailed instructions can be found in the [pantheon documentation for Git](https://docs.pantheon.io/guides/git/git-config#clone-your-site-codebase).

## Updating UCF Themes and Plugins

Almost all UCF themes and plugins are pulled into our Pantheon instances via [composer](https://getcomposer.org/). We publish our packages to a [private packagist repository](https://ucf.github.io/ucf-packagist/) the [[UCF Packagist Management|maintenance and upkeep]] of which is Web Communications responsibility. Generally speaking, when a theme or plugin requires a change, the code is developed locally on the developer's machine, and committed to the theme or plugin's repository. It goes through the normal [[Pull Requests|pull request]] process, and when it is ready to be released, a new tag/release is created and an automated process causes the new release to be published on the [[UCF Packagist Management|UCF packagist repository]].

To pull in that new version of the package, you use composer like you would with other PHP packages. So, for example, to update the `ucf/main-site-components` plugin, you'd need to run `composer update ucf/main-site-components` from within whatever Pantheon instance repository you wish to update. 

## Manually Updating Themes and Plugins

Sometimes it's necessary to manually update a theme or plugin, like when a premium plugin does not have a composer package. To do this, you simply copy in the files to the correct folder within the wp-content directory of the Pantheon instance repository, check to ensure the correct files have been added/updated/removed (using something like a `git diff` or `git status` command), and then commit the changes.
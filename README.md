# wfaa/php-config

Shared PHP coding standards, static-analysis config, and editor settings for
WFAA / Wisconsin Foundation php projects. One place to own the rules; every
project inherits them.

This repo ships **two installable packages** plus a set of copy-in templates:

| What                       | How it's distributed               | What it gives you                                                                                    |
| -------------------------- | ---------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `wfaa/php-config`          | Composer (private Bitbucket)       | PHPCS base ruleset, PHPStan base config, and the whole lint/analysis toolchain as transitive deps    |
| `@wfaa/prettier-config`    | npm (private Bitbucket)            | Shared Prettier config                                                                               |
| Editor + project templates | `vendor/bin/wfaa-init` copies them | `.vscode/`, `.zed/`, `phpcs.xml.dist`, `phpstan.neon.dist`, `.lintstagedrc.json`, `.prettierrc.json` |

Composer can version and auto-update the _rules_; it can't drop files into a
project root, so editor settings and the per-project stubs are **scaffolded once**
by `wfaa-init` and then tweaked per project.

## Quick start (fresh project)

The `composer.json` below is the exact setup **verified green** against
`github.com/lee-wfaa/wfaa-php-config`. It targets the public GitHub repo on the
`main` branch (no release tag yet), so it opts into dev stability for this one
package — `prefer-stable` keeps every other dependency on stable releases.

```json
{
	"minimum-stability": "dev",
	"prefer-stable": true,
	"repositories": [
		{ "type": "vcs", "url": "https://github.com/lee-wfaa/wfaa-php-config.git" }
	],
	"require-dev": {
		"wfaa/php-config": "dev-main"
	},
	"config": {
		"allow-plugins": {
			"dealerdirect/phpcodesniffer-composer-installer": true
		}
	}
}
```

Install, scaffold, and verify:

```bash
composer update wfaa/php-config
vendor/bin/wfaa-init                            # scaffold config + editor files
vendor/bin/phpcs                                # lint against the shared ruleset
vendor/bin/phpstan analyse --memory-limit=1G    # WordPress-aware static analysis
```

Once a release tag exists (`git tag v1.0.0 && git push --tags`) — or the repo
moves to private Bitbucket — drop the two stability flags and pin a version
instead of tracking the branch:

```json
{
	"repositories": [
		{ "type": "vcs", "url": "https://github.com/lee-wfaa/wfaa-php-config.git" }
	],
	"require-dev": {
		"wfaa/php-config": "^1.0"
	},
	"config": {
		"allow-plugins": {
			"dealerdirect/phpcodesniffer-composer-installer": true
		}
	}
}
```

> The `allow-plugins` entry must live in the **consuming** project's root
> `composer.json` — Composer won't run the PHPCS installer plugin from a
> dependency's config alone.

## Repo contents

```
phpcs-base.xml          # shared PHPCS ruleset (referenced from consumers' phpcs.xml.dist)
phpstan-base.neon       # shared PHPStan config (included by consumers' phpstan.neon.dist)
index.json              # the @wfaa/prettier-config payload (package.json main)
composer.json           # defines wfaa/php-config + pins the tool versions
package.json            # defines @wfaa/prettier-config
bin/wfaa-init           # copies templates/ into a project
templates/              # copy-in starters (phpcs, phpstan, editor settings, lint-staged)
```

## Consuming in a project

### 1. PHP standards (Composer)

Because this is a **private** Bitbucket repo, add it as a VCS repository in the
consuming project's `composer.json`:

```json
{
	"repositories": [
		{ "type": "vcs", "url": "git@bitbucket.org:CHANGEME/wfaa-php-config.git" }
	],
	"require-dev": {
		"wfaa/php-config": "^1.0"
	},
	"config": {
		"allow-plugins": {
			"dealerdirect/phpcodesniffer-composer-installer": true
		}
	}
}
```

> The `allow-plugins` entry is required in the **consuming** project's root
> `composer.json` — Composer will not run the PHPCS installer plugin from a
> dependency's config alone.

Then:

```bash
composer update wfaa/php-config
vendor/bin/wfaa-init            # scaffold phpcs.xml.dist, phpstan.neon.dist, editor configs
```

Point the project's own config files at the vendored bases (the templates already
do this):

```xml
<!-- phpcs.xml.dist -->
<rule ref="vendor/wfaa/php-config/phpcs-base.xml"/>
```

```neon
# phpstan.neon.dist
includes:
    - vendor/wfaa/php-config/phpstan-base.neon
```

Run the tools:

```bash
vendor/bin/phpcs                       # lint
vendor/bin/phpcbf                      # auto-fix
vendor/bin/phpstan analyse --memory-limit=1G
```

### 2. Prettier (npm)

Install the config package from Bitbucket and reference it:

```bash
npm install --save-dev "git+ssh://git@bitbucket.org/CHANGEME/wfaa-php-config.git"
```

`.prettierrc.json` (written by `wfaa-init`) is just:

```json
"@wfaa/prettier-config"
```

## Maintaining this repo

- **Versioning** — Composer and npm both resolve versions from **git tags**. Cut
  a `vX.Y.Z` tag when you change the rules; consumers move at their own pace via
  their version constraint (`^1.0`, etc.). Bump the major for a breaking rule
  change (anything that turns previously-passing code into an error).
- **Tool versions** live in `composer.json` `require`. Bump them here once and
  every project inherits the change on its next `composer update`.
- **Rule edits** go in `phpcs-base.xml` / `phpstan-base.neon`. Remember these
  affect every consuming project — per-project exceptions belong in the
  project's own `phpcs.xml.dist` / `phpstan.neon.dist`, not here.

## Before first use — replace the placeholders

- `CHANGEME` in this README, `composer.json` is implicit (VCS url lives in
  consumers), and `package.json` `repository.url` → your Bitbucket workspace/repo.

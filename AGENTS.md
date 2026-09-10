# blog

Zhengyu Chen's English-language technical writing and portfolio site, published at [prov1dence.top](https://prov1dence.top/).

> This file describes the active maintenance workflow. Do not restore historical deployment or content conventions.

## Stack

- **Generator:** Hugo v0.143.1 extended
- **Theme:** PaperMod at `themes/PaperMod`, the repository's only git submodule
- **Hosting:** GitHub Pages
- **Deployment:** a push to `master` triggers `.github/workflows/deploy.yml`, which builds the site and publishes the result to `gh-pages`
- **Configuration:** `config.yml`

## Content policy

- The public site is English-only. No Chinese text or Chinese-language attachments may be added under `content/`, `static/`, `config.yml`, or public-facing metadata.
- Current published writing focuses on AI agents, distributed systems, and software engineering.
- Every published article should include a specific `description` and cite primary sources for factual or version-sensitive claims.
- Historical Chinese material lives under `archive/zh/`, outside Hugo's publishing tree. Preserve archived source wording and structure.
- If archived material is substantially rewritten in English, publish it as a new article and add a Hugo alias for the old public URL when appropriate.

## Structure

```text
content/posts/          # English articles grouped by topic
content/about.md        # /about
static/attachments/     # public English-language resume and article assets
archive/zh/             # unpublished historical Chinese material
themes/PaperMod/        # theme submodule
layouts/                # project-level theme overrides
.github/workflows/      # CI deployment
public/                 # ignored local build output; never commit
```

## Common commands

```bash
hugo server -D      # local preview including drafts
hugo --minify       # local build verification only
```

## Deployment

The only supported deployment flow is to commit source changes to `master` and push the main repository. GitHub Actions builds and publishes the site to `gh-pages`, usually within about one minute.

```bash
GIT_SSH_COMMAND="ssh -p 443 -i ~/.ssh/id_ed25519 -o IdentitiesOnly=yes" git push origin master
```

Remote: `git@ssh.github.com:chr1sc2y/blog.git`.

Do not:

- commit `public/`;
- modify `gh-pages` by hand;
- push generated output to the archived `chr1sc2y/prov1dence.github.io` repository;
- edit files inside the PaperMod submodule for site-specific styling.

Place theme overrides in project-level `layouts/` or `assets/` so Hugo can resolve them before the submodule versions.

## Drafts

Pages with `draft: true` are excluded from production because CI builds without `-D`.

## Writing workflow

The active article workflow is `.cursor/skills/write-tech-article/SKILL.md`. New work must follow the English-only content policy above.

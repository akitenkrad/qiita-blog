# How it works

**English** | [日本語](how-it-works.ja.md)

Articles are written once, on Zenn. This repository holds the Qiita copy of the articles that are marked for Qiita, and posting to Qiita happens from here.

## The pipeline

A GitHub Actions workflow in the Zenn repository does everything. It runs on every push to the Zenn repository's `main` branch that touches articles, images or the conversion tool, and once a day at 09:13 JST so that articles whose waiting period has just ended are picked up even on days without a push.

1. **Convert.** The `zenn-tools qiita --apply` command reads every Zenn article whose frontmatter has `qiita: true`, and converts the ones that are ready into `public/<Zenn article ID>.md`.
2. **Push articles and images.** The converted articles and images are committed to this repository and pushed. Images go first so that they are already served when Qiita renders the article.
3. **Post.** Qiita CLI (`qiita publish --all`) posts new articles and updates the ones whose content changed.
4. **Record the Qiita IDs.** Qiita CLI writes the issued article `id` and `updated_at` into each file in `public/`. The workflow commits these back here. This step runs even when posting partly failed: an ID that is not recorded would make the next run post the same article again.

## When an article is converted

An article is written to `public/` only when all of these hold:

- It is published on Zenn (`published: true`), and its Zenn publication date plus the configured waiting period (3 days by default) has passed.
- It has a title and 1–5 topics. Zenn topics become Qiita tags.
- Every image it uses exists.

An article whose title, tags and body did not change is not rewritten, so Qiita is not updated needlessly.

## Conversion

Each article starts with a note linking to the original on Zenn. Zenn-specific syntax is rewritten to its Qiita equivalent:

- `:::message` boxes become `:::note` boxes, and `:::details` becomes `<details>`.
- Display math becomes a `math` code block.
- Embeds (`@[card]`, `@[github]`, `@[youtube]`, …) become bare URLs, which Qiita turns into link cards.
- Images are pointed at this repository (see below).

Code blocks are left as they are.

## Images

Images are served from this repository through `raw.githubusercontent.com`. Each file name carries a hash of the image content, such as `images/<article>/<name>-<hash>.png`. A replaced image therefore gets a new URL, and Qiita shows the new image regardless of any cache. Old images are kept because earlier versions of an article refer to them.

## Do not edit this repository by hand

- **Articles in `public/`** are overwritten from Zenn on the next change. An edit made directly on Qiita also makes Qiita CLI refuse the next update, because the copy here would then be older than the article on Qiita.
- **The `id` lines in `public/`** link each file to its Qiita article. Losing one makes the next run post a duplicate article.
- **The workflow is the only writer.** Keeping a single writer avoids mixing up article IDs between concurrent pushes.

To change an article, edit it on Zenn. To withdraw an article from Qiita, unpublish or delete it on Qiita yourself: unpublishing on Zenn or removing `qiita: true` does not remove the Qiita copy.

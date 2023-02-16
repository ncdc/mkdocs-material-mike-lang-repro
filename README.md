# Example repo for mkdocs-material + mike + multiple languages

This is an example repo for reproducing some issues I've encountered trying to use mkdocs-material, `mike` for 
multiple version support, and i8n.

I'm trying to use this for `docs.kcp.io/kcp` and we'd like the following URL structure:

- `docs.kcp.io/kcp/<version>/<language>/...`

## Structure

This is meant to work with a repo that contains code + docs. All docs live in `docs`.

`docs/config` contains `mkdocs.yml` files, 1 per language, such as `docs/config/en/mkdocs.yml`.

- `site_url` is set to e.g. `https://docs.kcp.io/kcp`. Ideally this would be something more like
  `https://docs.kcp.io/kcp/{{ version }}/{{ language }}` or something along those lines.
- `extra.alternate[*].link` ideally should not be needed, or be made optional. The language switcher URL should 
   be `{{ base_url }}/{{ version }}/{{ language }}/{{ current page path }}`

`docs/content` contains per-language content, such as `docs/content/en`.

`docs/generated/branch` is where `mkdocs build` outputs what it builds.

`docs/mkdocs.yml` is just for `mike`. Its `docs_dir` is `generated/branch`, so it deploys everything generated above.

## Workflow

1. Run `mkdocs build` for each language. Output will go to `docs/generated/branch/<language>`
2. After you've built all languages, run `mike deploy --config-file docs/mkdocs.yml`

## Issues

1. The TypeScript that queries `versions.json` assumes that it's located one level up from `base_url`. For a 
   URL such as `docs.kcp.io/kcp/main/en/index.html`, that translates to `docs.kcp.io/kcp/main/versions.json`, which 
   results in a 404. The file actually lives at `docs.kcp.io/kcp/versions.json`. `mike` is configured to deploy to 
   `docs.kcp.io/kcp`, so it places `versions.json` there. The problem arises because my language's `mkdocs.yml` has 
   a `site_dir` of `../../generated/branch/en` but `mike` is deploying `generated/branch`. This adds `en` into the path
   but when I run `mkdocs build config/en/mkdocs.yml`, it doesn't know that it will end up being served one level 
   deeper. This is where being able to set `site_url` to `https://docs.kcp.io/kcp/{{ version }}/{{ language }}` 
   would be helpful.
2. The site language selector in `header.html` doesn't seem to allow any dynamic content. It would be really helpful 
   if it were either version aware, or it was possible to customize the href without needing to override the entire 
   `header.html` partial.
3. The generated canonical URL in the HTML for all pages is missing both the version and the language. For example,
   `https://docs.kcp.io/kcp/main/en/developers/replicate-new-resource/` has
```html
<link rel="canonical" href="https://docs.kcp.io/kcp/developers/replicate-new-resource/">
```

## Workarounds
1. I wrote a [plugin](https://github.com/ncdc/mkdocs-modify-base-url) that lets you customize the `base_url` before
   content is rendered. This lets me fix the dynamic version retrieval to account for the extra path segment. I added 
   this to the `plugins` section in my language `mkdocs.yml`:

```yaml
- modify-base-url:
  prefix: '../'
```

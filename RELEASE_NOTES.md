# v0.0.46-patch-1: Add `pages` variable to per-page template context

## Summary

This patch adds the `pages` variable to the template context when rendering individual pages, enabling index pages for sections (blog, tags, recent news, etc.) to iterate over and preview other pages.

## Changes

**Fix:** Add `pages` navigation collection to per-page template rendering

- **File:** `crates/zensical/src/structure/page.rs`
- **Change:** Collect pages from the navigation tree and pass them to the template context
- **Impact:** Templates can now access `{{ pages }}` to build section index pages, archive listings, tag clouds, etc.

## Code Change

```rust
// In page.rs, line ~210
let pages = nav.iter().collect::<Vec<_>>();

let output = template.render_with_context(context! {
    generator => GENERATOR,
    nav => nav,
    pages => pages,  // NEW: Available in all per-page templates
    base_url => config.get_base_url(&self.url),
    extra_css => config.project.extra_css.clone(),
    extra_javascript => config.project.extra_javascript.clone(),
    config => config.project.clone(),
    tags => self.tags(),
    page => self,
})?;
```

## Rationale

The upstream Zensical project has `pages` available only in static template rendering (sitemap, 404 pages) but not in per-page rendering. This patch extends it to per-page context without significant performance impact—collecting references is O(n) but negligible compared to Markdown parsing and I/O.

## Usage in Templates

You can now iterate over all pages in your templates:

```jinja2
<div class="page-list">
  {% for page in pages %}
    <a href="{{ page.url }}">{{ page.title }}</a>
  {% endfor %}
</div>
```

Perfect for building:
- Blog index pages
- Tag archive pages
- Recent news sections
- Sitemap-like indexes

## Upstream Status

This is a temporary patch for local use until the fix is merged upstream. Once merged, switch back to the official package.

## Installation

### GitHub Actions (Ubuntu)

```yaml
- run: |
    curl -L https://github.com/codeshell/zensical/releases/download/v0.0.46-patch-1/zensical-0.0.46-cp310-abi3-manylinux_2_17_x86_64.manylinux2014_x86_64.whl -o /tmp/zensical.whl
    uv pip install /tmp/zensical.whl
- run: uv run zensical build --clean
```

### Local Development (Windows)

```powershell
uv pip install https://github.com/codeshell/zensical/releases/download/v0.0.46-patch-1/zensical-0.0.46-cp310-abi3-win_amd64.whl
```

## Assets

- `zensical-0.0.46.tar.gz` — Source distribution
- `zensical-0.0.46-cp310-abi3-manylinux_2_17_x86_64.manylinux2014_x86_64.whl` — Pre-compiled wheel (Linux)
- `zensical-0.0.46-cp310-abi3-win_amd64.whl` — Pre-compiled wheel (Windows)

---
title: Generic Folder Omnisearch Search Bar
created: 2026-05-01
modified: 2026-05-01
tags:
  - template
  - dataviewjs
  - omnisearch
---
# Generic Folder Omnisearch Search Bar

Copy this block into any folder note and change only `FOLDER`.

```dataviewjs
const FOLDER = "PUT/FOLDER/PATH/HERE/", MAX = 12;
const root = dv.el("div", "", { cls: "folder-omni-search" });
root.createEl("style", { text: `
.folder-omni-search{width:min(100%,var(--file-line-width,700px));max-width:var(--file-line-width,700px);margin:.5rem auto .85rem;padding:.38rem .7rem;border:1px solid var(--background-modifier-border);border-radius:var(--radius-m);background:color-mix(in srgb,var(--background-secondary) 72%,transparent);color:var(--text-normal)}
.folder-omni-search input{width:100%;background:transparent;color:var(--text-normal);border:0;border-radius:0;padding:.22rem .1rem;font:inherit}
.folder-omni-search input:focus{box-shadow:none;outline:none}
.folder-omni-search .omni-meta{color:var(--text-faint);font-size:var(--font-ui-smaller);margin-top:.25rem}
.folder-omni-search .omni-results{margin-top:.25rem}
.folder-omni-search .omni-result{display:flex;justify-content:space-between;gap:.75rem;padding:.25rem .1rem;border-bottom:1px solid var(--background-modifier-border-hover);cursor:pointer}
.folder-omni-search .omni-result:hover .omni-title{color:var(--link-color-hover)}
.folder-omni-search .omni-title{color:var(--link-color);overflow:hidden;text-overflow:ellipsis;white-space:nowrap}
.folder-omni-search .omni-date{color:var(--text-faint);flex:0 0 auto;font-size:var(--font-ui-smaller)}
` });
const input = root.createEl("input", { attr: { type: "search", placeholder: "Search…", spellcheck: "false" } });
const results = root.createDiv({ cls: "omni-results" });

const omni = () => app.plugins?.plugins?.omnisearch?.api ?? globalThis.omnisearch;
const msg = text => (results.empty(), results.createDiv({ cls: "omni-meta", text }));
const date = path => {
  const p = dv.page(path), raw = p?.created ?? p?.file?.ctime;
  return raw ? dv.date(raw).toFormat("MMM d, yyyy") : "—";
};
const debounce = (fn, ms = 220) => { let t; return (...a) => (clearTimeout(t), t = setTimeout(() => fn(...a), ms)); };

const render = hits => {
  results.empty();
  if (!hits.length) return msg("No results.");
  for (const r of hits.slice(0, MAX)) {
    const row = results.createDiv({ cls: "omni-result" });
    row.createDiv({ cls: "omni-title", text: r.basename ?? r.path.split("/").pop().replace(/\.md$/, "") });
    row.createDiv({ cls: "omni-date", text: date(r.path) });
    row.onclick = e => app.workspace.openLinkText(r.path, "", e.ctrlKey || e.metaKey);
  }
};

input.oninput = debounce(async () => {
  const query = input.value.trim(), api = omni();
  if (!query) return results.empty();
  if (!api?.search) return msg("Omnisearch unavailable.");
  try { render((await api.search(query)).filter(r => r.path?.startsWith(FOLDER))); }
  catch (e) { console.error(e); msg("Search failed."); }
});
```


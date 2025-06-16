This site is based on the Chirpy Jekyll theme and can be found at [Cotes Chung repository](https://github.com/cotes2020/jekyll-theme-chirpy.git).

**Adding Notes and Project Section**
1. Created _notes/ and _projects/ folders (just like _posts/) to store note and project markdown files.

2. Added a new file notes.md (and projects.md) in the _tabs/ folder with the following front matter:

```yaml
---
title: Notes
icon: fas fa-stream
order: 5
layout: category
category: notes
---
```
3. Set the categories front matter in each post to match (notes or projects), so they appear under their respective tabs.

4. Updated index.html to exclude notes and projects from the homepage blog feed.




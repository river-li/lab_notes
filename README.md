---
title: Example
mathjax: true
author: Admin
date: 2019-10-11 19:44:05
categories: 
- Paper
---

# Introduction

Add your Paper Notes as markdown files, which should have a header like such format:

```
title: Paper's Title
mathjax: true
author: Your name
date: 2019-10-11 19:44:05
categories: 
- Paper
```

## Math Formula

Pay attention that if you have used latex mathjax additions, the mathjax above must set to True.

Just like the header of this file.

## Insert images

When you want to insert a image in your post, make a new directory in the repository, which should have excetly the same name as your markdown file, except for the file suffix.

For example, the directories tree should look like that:

```
tmp
├── example
│   ├── 1.png
│   └── 2.png
└── example.md
```

Copy your images to the direcory same with your post name.

And in your markdown file, you also need to add some codes.

```markdown
{% asset_img 1.png 1.png %}
```

Don't use traditional markdown image hyper link, or the image will not display correctly.

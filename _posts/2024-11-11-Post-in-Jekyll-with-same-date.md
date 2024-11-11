---
title: Posts in Jekyll with same date
date: 2024-11-11 11:05:54 +0200
---

As you should know, post on Jekyll is equal to create a new `.md` file in `_posts` folder.

Then you name it as `YEAR-MONTH-DAY-title.MARKUP`
Where YEAR is a four-digit number, MONTH and DAY are both two-digit numbers, and MARKUP is the file extension representing the format used in the file, `md` for Markdown. For example, the following are examples of valid post filenames:

```
2024-08-01-article-1.md
2024-08-01-article-2.md
```

By default, Jekyll uses the date you use in the filename, so 2024-08-01-article-2.md is read by Jekyll, and then when it displays the posts, it uses that date.

However, your post should also have a date field in the YAML front matter. You will find that when you create a new blank Jekyll site, it includes that field. This is a best practice, especially for people like you who may post more than once a day and care about how the file gets displayed.

Using the date field, you can use this format:
```
date: 2024-08-01
```

But it's importat to include the timezone

![](/assets/images/blog/date_frontmatter_jekyll.png)


In our case, that does not help a lot because you want more granularity, so you can extend the date format like this, which includes `HH:MM;SS +TTTT`
```
date: 2024-08-01 02:05:54 +0200
```

Here is what you should minimally do so you get the sorting right, assuming article 1 was written first and article 2 was written second:

2024-08-01-article-1.md:
```
---
layout: single
title: article-1
date: 2024-08-01 02:01:00 +0200
excerpt:
seo_title:
seo_description:
categories:
tags:
---
The content goes here...
```

2024-08-01-article-2.md:
```
---
layout: single
title: article-2
date: 2024-08-01 02:02:00 +0200
excerpt:
seo_title:
seo_description:
categories:
tags:
---
The content goes here...
```
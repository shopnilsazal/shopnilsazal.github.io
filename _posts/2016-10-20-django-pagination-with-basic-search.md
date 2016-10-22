---
layout: post
title: Django Pagiantion with Basic Search
description: "Number based pagination with basic search in django."
modified: 2016-06-01T15:27:45-04:00
tags: [django, pagination, search]
image:
  feature: blog-banner-1.jpg
---

In a web application, search and pagination are common things. Django offers a few classes to manage query and paginated data. In this post I will write about how you can create a basic search and number based pagination.

Let's assume we have a `Post` model with some basic fields like title, content, user(foreign key), publish date, updated date etc. 

## Search

Now we will create search form. In your `post_list.html` template create a search from like this.

```html
{% raw %} 
<form method="GET" >
    <div class="input-field">
        <input type="text" name="q" value="{{ request.GET.q }}" placeholder="Search Here...">
       <button class="btn" type="submit" name="action">Search</button>
    </div>
</form>
{% endraw %}
```

To get search form working we can use `Q` query related class. If you don't know about `Q`, Here is a quote from official documentation.

> In general, Q() objects make it possible to define and reuse conditions. This permits the construction of complex database queries using `|` (OR) and `&` (AND) operators; in particular, it is not otherwise possible to use OR in QuerySets.

We can write this code snippet in `views.py`.

```python
from django.shortcuts import render
from django.db.models import Q
from .models import Post

def post_list(request):
    posts_list = Post.objects.all()
    query = request.GET.get('q')
    if query:
        posts_list = Post.objects.filter(
            Q(title__icontains=query) | Q(content__icontains=query) |
            Q(user__first_name__icontains=query) | Q(user__last_name__icontains=query)
        ).distinct()
    
    context = {
        'posts': posts_list
    }
    return render(request, "post-list.html", context)
```

Here `icontains` means "Case-insensitive containment". Now, you can display posts in `posts_list.html` template like this.

```html
{% raw %} 
{% for post in posts %}

    <span class="meta">Posted on: {{ post.updated|date }}</span>
    <h3 class="title">{{ post.title }}</h3>
    <p>{{ post.content|truncatechars:140 }}</p>

{% endfor %}
{% endraw %}
```


## Pagination

We will use `Paginator` class for pagination and `EmptyPage, PageNotAnInteger` for exception handling. Paginator class lives in `django.core.paginator`. Here is the full `post_list` view in `views.py`.

```python
from django.shortcuts import render
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger
from django.db.models import Q
from .models import Post

def post_list(request):
    posts_list = Post.objects.all()
    query = request.GET.get('q')
    if query:
        posts_list = Post.objects.filter(
            Q(title__icontains=query) | Q(content__icontains=query) |
            Q(user__first_name__icontains=query) | Q(user__last_name__icontains=query)
        ).distinct()
    paginator = Paginator(posts_list, 6) # 6 posts per page
    page = request.GET.get('page')

    try:
        posts = paginator.page(page)
    except PageNotAnInteger:
        posts = paginator.page(1)
    except EmptyPage:
        posts = paginator.page(paginator.num_pages)

    context = {
        'posts': posts
    }
    return render(request, "post-list.html", context)
```

Now we will add pagination in `post-list.html` template. Append this code snippet in `post_list.html` template.

```html
{% raw %}
{% if posts.has_other_pages %}
    <ul class="pagination">
        {% if posts.has_previous %}
            <li class="waves-effect"><a href="?page=
                    {{ posts.previous_page_number }}{% if request.GET.q %}&q={{ request.GET.q }}{% endif %}"><i
                    class="material-icons">chevron_left</i></a></li>
        {% else %}
            <li class="disabled"><a href="#!"><i class="material-icons">chevron_left</i></a></li>
        {% endif %}
        {% for num in posts.paginator.page_range %}
            {% if posts.number == num %}
                <li class="active"><a href="#!">{{ num }}</a></li>
            {% else %}
                <li class="waves-effect"><a
                        href="?page={{ num }}{% if request.GET.q %}&q={{ request.GET.q }}{% endif %}">{{ num }}</a></li>
            {% endif %}
        {% endfor %}
        {% if posts.has_next %}
            <li class="waves-effect"><a
                    href="?page={{ posts.next_page_number }}{% if request.GET.q %}&q={{ request.GET.q }}{% endif %}"><i
                    class="material-icons">chevron_right</i></a></li>
        {% else %}
            <li class="disabled"><a href="#!"><i class="material-icons">chevron_right</i></a></li>
        {% endif %}
    </ul>
{% endif %}
{% endraw %}
```

N.B: This pagination is based on [Materializecss Pagination](http://materializecss.com/pagination.html). You can follow this for Bootstrap. 


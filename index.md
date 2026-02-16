---
layout: default
---

{% assign featured = site.posts | where: "featured", true %}
{% assign regular = site.posts | where_exp: "post", "post.featured != true" %}

<div>
    <div class="mb-8">
        {% for post in featured %}
        <article class="rounded-xl bg-surface dark:bg-surface-dark p-6 shadow-lg md:flex md:gap-6 hover:shadow-xl transition-shadow">
            <div class="md:w-1/2 mb-4 md:mb-0">
                <a href="{{ post.url }}">
                    <figure class="post-cover">
                        <picture>
                        </picture>
                    </figure>
                </a>
            </div>
            <div class="md:w-1/2 flex flex-col justify-center">
                <span class="text-sm font-medium text-indigo-400 uppercase tracking-wide mb-2">Featured</span>
                    <h2 class="text-3xl font-bold text-text-primary dark:text-text-primary-dark leading-snug mb-3">
                        <a href="{{ post.url }}" class="hover:underline">{{ post.title }} </a>
                    </h2>
                    <p class="text-text-secondary dark:text-text-secondary-dark text-base">
                        {{ post.summary }}
                    </p>
            </div>
        </article>
        {% endfor %}
    </div>
</div>

---
layout: default
---

{% assign featured = site.posts | where: "featured", true %}
{% assign posts = site.posts | where_exp: "post", "post.featured != true" %}

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
    <div class="grid grid-cols-1 lg:grid-cols-4 gap-6">
        <div class="lg:col-span-3 grid grid-cols-1 md:grid-cols-2 gap-6">
            {% for post in posts %}
            <article>
                <figure class="post-cover">
                    <picture>
                    </picture>
                </figure>
                <h2 class="text-2xl font-semibold mt-4 text-text-primary dark:text-text-primary-dark mb-2">
                    <a href="{{ post.url }}" class="hover:underline">{{ post.title }}</a>
                </h2>
                {% include post-categories-list.html post=post %}
                <p class="mt-4 text-gray-300 text-text-secondary dark:text-text-secondary-dark">Learn how to add SEO and Open Graph metadata to your Hugo site for better sharing and indexing.</p>
                <div class="hidden sm:flex flex-wrap gap-2 mt-4">
                    {% for tag in post.tags %} 
                    <a href="/tags/{{ tag }}" class="text-sm bg-gray-800 text-gray-200 px-2 py-1 rounded hover:bg-gray-700">{{ tag }}</a>
                    {% endfor %}
                </div>
            </article>
            {% endfor %}
        </div>
    </div>
</div>

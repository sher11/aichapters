---
layout: default
title: Home
---

# Welcome to AIChapters

Learn Artificial Intelligence and Machine Learning through structured, chapter-based courses.

## Available Courses

<div class="course-list">
  {% assign course_pages = site.pages | where_exp: "page", "page.path contains 'courses/'" | where_exp: "page", "page.name == 'index.md'" | sort: "title" %}

  {% for course in course_pages %}
    <div class="course-card">
      <h3>{{ course.title }}</h3>
      <p>{{ course.description }}</p>
      <a href="{{ site.baseurl }}{{ course.url }}">Start Course &rarr;</a>
    </div>
  {% endfor %}
</div>

## How It Works

1. Choose a course from the list above
2. Navigate through chapters using the sidebar
3. Use the previous/next buttons at the bottom of each chapter
4. Learn at your own pace!

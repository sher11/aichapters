---
layout: default
title: Home
---

# Staff-Level FAANG Interview Prep

Master data structures, algorithms, system design, and coding patterns for L6/E6 technical interviews at top tech companies.

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

## Format

- **Cheatsheet-style:** Short, crisp chapters for quick revision
- **High-frequency:** Focus on commonly asked patterns and problems
- **Trade-offs:** Discuss when to use one approach vs another
- **Q&A sections:** Test your understanding with focused questions
- **Staff-level:** L6/E6 depth with system design considerations

## How to Use

1. Choose a course from the list above
2. Navigate through chapters using the sidebar
3. Review main concepts, then test with Q&A chapters
4. Focus on trade-offs and real-world applications
5. Revise frequently using the cheatsheet format

---
layout: page
permalink: /repositories/
title: Repositories
description: Git profile and repository showcase.
nav: true
nav_order: 5
---

{% if site.data.repositories.github_users %}

## GitHub Profile

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% for user in site.data.repositories.github_users %}
    {% include repository/repo_user.liquid username=user %}
  {% endfor %}
</div>

---

{% if site.repo_trophies.enabled %}
{% for user in site.data.repositories.github_users %}
{% if site.data.repositories.github_users.size > 1 %}

  <h4>{{ user }}</h4>
  {% endif %}
  <div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo_trophies.liquid username=user %}
  </div>

---

{% endfor %}
{% endif %}
{% endif %}

{% if site.data.repositories.external_contributions %}

## Open Source Contributions

{% for contrib in site.data.repositories.external_contributions %}
<div class="card mt-3">
  <div class="card-body">
    <h3 class="card-title">
      <a href="{{ contrib.repo_url }}" target="_blank">
        <i class="fab fa-github"></i> {{ contrib.repo }}
      </a>
    </h3>
    <p class="card-text">{{ contrib.description }}</p>
    
    <a href="https://github.com/{{ contrib.repo }}/pulls?q=is%3Apr+author%3Arothej" 
       target="_blank" 
       class="btn btn-primary mt-3">
      <i class="fas fa-code-branch"></i> View My Pull Requests
    </a>
  </div>
</div>
{% endfor %}

---

{% endif %}

{% if site.data.repositories.github_repos %}

## GitHub Repositories

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% for repo in site.data.repositories.github_repos %}
    {% include repository/repo.liquid repository=repo %}
  {% endfor %}
</div>
{% endif %}
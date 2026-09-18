---
layout: page
permalink: /publications/
title: publications
description: Papers.
years: [2026,2025,2024,2023,2022,2021,2020]
nav: true
nav_order: 1
---

Most recent list at [Google scholar](https://scholar.google.co.uk/citations?hl=en&user=CdpLhlgAAAAJ). * denote equal contribution, ⁺ denotes equal contribution with random order.


<!-- _pages/publications.md -->
<div class="publications">
  <div class="publication-browser">
    <p class="publication-browser__intro">
      Explore publications by research topic. Topics overlap, so a paper may appear in more than one collection.
    </p>
    <div class="publication-filters" role="group" aria-label="Filter publications by topic">
      <button type="button" class="publication-filter is-active" data-topic="all" aria-pressed="true" aria-controls="publication-results">
        All <span class="publication-filter__count" data-topic-count="all"></span>
      </button>
      <button type="button" class="publication-filter" data-topic="self-supervised" aria-pressed="false" aria-controls="publication-results">
        Self-Supervised Learning <span class="publication-filter__count" data-topic-count="self-supervised"></span>
      </button>
      <button type="button" class="publication-filter" data-topic="multimodal" aria-pressed="false" aria-controls="publication-results">
        Multimodal Learning <span class="publication-filter__count" data-topic-count="multimodal"></span>
      </button>
      <button type="button" class="publication-filter" data-topic="llms" aria-pressed="false" aria-controls="publication-results">
        LLMs <span class="publication-filter__count" data-topic-count="llms"></span>
      </button>
      <button type="button" class="publication-filter" data-topic="post-training" aria-pressed="false" aria-controls="publication-results">
        Post-Training <span class="publication-filter__count" data-topic-count="post-training"></span>
      </button>
      <button type="button" class="publication-filter" data-topic="generative" aria-pressed="false" aria-controls="publication-results">
        Generative Models <span class="publication-filter__count" data-topic-count="generative"></span>
      </button>
      <button type="button" class="publication-filter" data-topic="3d-video" aria-pressed="false" aria-controls="publication-results">
        3D &amp; Video <span class="publication-filter__count" data-topic-count="3d-video"></span>
      </button>
      <button type="button" class="publication-filter" data-topic="efficient" aria-pressed="false" aria-controls="publication-results">
        Efficient Learning <span class="publication-filter__count" data-topic-count="efficient"></span>
      </button>
      <button type="button" class="publication-filter" data-topic="trustworthy" aria-pressed="false" aria-controls="publication-results">
        Trustworthy AI <span class="publication-filter__count" data-topic-count="trustworthy"></span>
      </button>
    </div>
  </div>

  <div class="publication-results-header">
    <h2 class="category" id="publication-results-heading">All Publications</h2>
    <p class="publication-filter-status" id="publication-filter-status" aria-live="polite"></p>
  </div>

  <div id="publication-results">
    {%- for y in page.years %}
      <section class="publication-year-group" data-publication-year="{{ y }}">
        <h2 class="year">{{y}}</h2>
        {% bibliography -f papers -q @*[year={{y}}]* %}
      </section>
    {% endfor %}
  </div>

  <button type="button" class="show-more" id="show-more-btn" aria-controls="publication-results">
    Show More
  </button>

</div>

<script>
document.addEventListener('DOMContentLoaded', function() {
  const publicationContainer = document.querySelector('.publications');
  if (!publicationContainer) return;

  const showMoreBtn = document.getElementById('show-more-btn');
  const status = document.getElementById('publication-filter-status');
  const resultsHeading = document.getElementById('publication-results-heading');
  const filterButtons = Array.from(publicationContainer.querySelectorAll('.publication-filter'));
  const yearGroups = Array.from(publicationContainer.querySelectorAll('.publication-year-group'));
  const publicationItems = Array.from(
    publicationContainer.querySelectorAll('.publication-year-group .bibliography > li')
  );
  const pageSize = 30;
  let activeTopic = 'all';
  let visibleLimit = pageSize;

  function topicsFor(item) {
    const publicationEntry = item.querySelector('.publication-entry');
    return ((publicationEntry && publicationEntry.dataset.topics) || '')
      .split(',')
      .map(function(topic) { return topic.trim(); })
      .filter(Boolean);
  }

  function matchingItems() {
    if (activeTopic === 'all') return publicationItems;
    return publicationItems.filter(function(item) {
      return topicsFor(item).includes(activeTopic);
    });
  }

  function updateCounts() {
    publicationContainer.querySelectorAll('[data-topic-count]').forEach(function(count) {
      const topic = count.dataset.topicCount;
      const total = topic === 'all'
        ? publicationItems.length
        : publicationItems.filter(function(item) { return topicsFor(item).includes(topic); }).length;
      count.textContent = total;
    });
  }

  function updateResults() {
    const matches = matchingItems();
    const visibleItems = matches.slice(0, visibleLimit);
    const visibleSet = new Set(visibleItems);
    const activeButton = filterButtons.find(function(button) {
      return button.dataset.topic === activeTopic;
    });
    const topicLabel = activeButton
      ? activeButton.childNodes[0].textContent.trim()
      : 'All';

    publicationItems.forEach(function(item) {
      const isVisible = visibleSet.has(item);
      item.hidden = !isVisible;
      item.classList.toggle('is-visible', isVisible);
    });

    yearGroups.forEach(function(group) {
      const groupItems = Array.from(group.querySelectorAll('.bibliography > li'));
      group.hidden = !groupItems.some(function(item) { return visibleSet.has(item); });
    });

    filterButtons.forEach(function(button) {
      const isActive = button.dataset.topic === activeTopic;
      button.classList.toggle('is-active', isActive);
      button.setAttribute('aria-pressed', String(isActive));
    });

    resultsHeading.textContent = activeTopic === 'all' ? 'All Publications' : topicLabel;
    status.textContent = matches.length === 0
      ? 'No publications found for ' + topicLabel + '.'
      : 'Showing ' + visibleItems.length + ' of ' + matches.length + ' publications';

    showMoreBtn.style.display = visibleItems.length < matches.length ? 'inline-flex' : 'none';
  }

  function selectTopic(topic, updateUrl) {
    const topicExists = filterButtons.some(function(button) {
      return button.dataset.topic === topic;
    });
    if (!topicExists) topic = 'all';

    activeTopic = topic;
    visibleLimit = pageSize;
    updateResults();

    if (updateUrl && window.history && window.history.replaceState) {
      const url = new URL(window.location.href);
      if (topic === 'all') {
        url.searchParams.delete('topic');
      } else {
        url.searchParams.set('topic', topic);
      }
      window.history.replaceState({}, '', url);
    }
  }

  filterButtons.forEach(function(button) {
    button.addEventListener('click', function() {
      selectTopic(button.dataset.topic, true);
    });
  });

  showMoreBtn.addEventListener('click', function() {
    visibleLimit += pageSize;
    updateResults();
  });

  updateCounts();
  const requestedTopic = new URLSearchParams(window.location.search).get('topic');
  selectTopic(requestedTopic || 'all', false);
});
</script>

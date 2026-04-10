---
layout: categories
title: "Categories"
permalink: /categories/
---

<style>
.cat-nav {
  display: flex; gap: 8px; flex-wrap: wrap;
  margin-bottom: 24px; padding-top: 1rem;
}
.cat-btn {
  padding: 6px 16px; border-radius: 20px;
  border: 1px solid #e0e0e0;
  background: #fff; color: #555;
  font-size: 13px; cursor: pointer;
  transition: all 0.15s ease;
}
.cat-btn:hover { background: #f5f5f5; }
.cat-btn.active {
  background: #222; color: #fff;
  border-color: #222;
}
.post-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 12px;
}
.post-card {
  padding: 16px; border-radius: 12px;
  border: 1px solid #e0e0e0;
  background: #fff;
  text-decoration: none; color: inherit;
  display: block;
  transition: transform 0.15s ease, box-shadow 0.15s ease;
}
.post-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 16px rgba(0,0,0,0.08);
}
.post-card.hidden { display: none; }
.post-tag {
  display: inline-block; font-size: 11px;
  padding: 2px 8px; border-radius: 10px;
  background: #f0f0f0; color: #666;
  margin-bottom: 8px;
}
.post-title { font-size: 14px; font-weight: 600; margin: 0 0 4px; }
.post-date { font-size: 12px; color: #999; }
.count-label { font-size: 13px; color: #888; margin-bottom: 12px; }
</style>

<div class="cat-nav" id="catNav">
  <button class="cat-btn active" data-cat="all">📋 All</button>
  <button class="cat-btn" data-cat="dev">💻 개발</button>
  <button class="cat-btn" data-cat="ctf">🔐 CTF/Wargame</button>
  <button class="cat-btn" data-cat="bugbounty">🐞 BugBounty</button>
  <button class="cat-btn" data-cat="blog">📝 블로그/기술문서</button>
  <button class="cat-btn" data-cat="research">📚 논문/컨퍼런스</button>
  <button class="cat-btn" data-cat="career">🏆 공모전/자격증</button>
</div>

<p class="count-label" id="countLabel"></p>

<div class="post-grid" id="postGrid">
  {% for post in site.posts %}
  <a class="post-card" href="{{ post.url }}" data-cat="{{ post.categories[0] | downcase }}">
    <span class="post-tag">{{ post.categories[0] }}</span>
    <p class="post-title">{{ post.title }}</p>
    <p class="post-date">{{ post.date | date: "%Y-%m-%d" }}</p>
  </a>
  {% endfor %}
</div>

<script>
const catMap = {
  all: '전체', dev: '개발', ctf: 'CTF/Wargame',
  bugbounty: 'BugBounty', blog: '블로그/기술문서',
  research: '논문/컨퍼런스', career: '공모전/자격증'
};
const btns = document.querySelectorAll('.cat-btn');
const cards = document.querySelectorAll('.post-card');
const label = document.getElementById('countLabel');

function filter(cat) {
  let count = 0;
  cards.forEach(card => {
    const show = cat === 'all' || card.dataset.cat === cat;
    card.classList.toggle('hidden', !show);
    if (show) count++;
  });
  label.textContent = cat === 'all'
    ? `전체 ${count}개 글`
    : `${catMap[cat] || cat} ${count}개 글`;
}

btns.forEach(btn => {
  btn.addEventListener('click', () => {
    btns.forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
    filter(btn.dataset.cat);
  });
});

filter('all');
</script>

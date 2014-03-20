---
layout: home
---

<div class="index-content blog">
    <div class="section">
        <div class="cate-bar"><span id="cateBar"></span></div>

        <ul class="artical-list">
        {% for post in site.categories.blog %}
            <li>
                <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
                <div class="title-desc">{{ post.description }}</div>
            </li>
        {% endfor %}
        </ul>
    </div>
    <div class="aside" style="background-image: url('http://liyu1981.smugmug.com/Photography/365-days-365-photos/i-QCTQNpn/0/X3/IMG_3940-X3.jpg')">
      <div class="avatar circle" style="width: 150px; height: 150px; position: absolute; right: -75px; top: 75px;">
        <div class="center" style="margin-top: 4px; height: 142px; width: 142px; border-radius: 71px; background-image: url('https://secure.gravatar.com/avatar/6a1f8a9d412c8e54bed896135c6f7d0c?s=142')"></div>
      </div>
    </div>
</div>

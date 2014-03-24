---
layout: home
---

<div class="index-content blog">
    <div class="section">
        <div class="cate-bar">
          <a href="http://www.linkedin.com/pub/yu-li/6/3a4/499"><i class="icon-linkedin-sign icon-large"></i><span> Profile</span></a>
          <a href="http://www.github.com/liyu1981?tab=repositories"><i class="icon-github icon-large"></i><span> Code</span></a>
        </div>
        <ul class="artical-list">
        {% for post in site.categories.blog %}
            <li><div class="title-date">{{ post.date | date:"%Y-%m-%d" }}</div>
                <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
                <div class="title-desc">{{ post.description }}</div>
            </li>
        {% endfor %}
        </ul>
    </div>
    <script>
      $(function() {
        function geturl() {
          var all = [
            'url("http://liyu1981.smugmug.com/Photography/365-days-365-photos/i-QCTQNpn/0/X3/IMG_3940-X3.jpg")',
            'url("http://liyu1981.smugmug.com/Photography/365-days-365-photos/i-zvPmxS6/0/X3/IMG_3006-X3.jpg")',
            'url("http://liyu1981.smugmug.com/Photography/365-days-365-photos/i-mt6dbQD/0/X3/IMG_5953-X3.jpg")',
            'url("http://liyu1981.smugmug.com/Photography/365-days-365-photos/i-BW2MzfC/0/X3/IMG_3402-X3.jpg")',
            'url("http://liyu1981.smugmug.com/Photography/365-days-365-photos/i-HwZrx3V/0/X3/IMG_3374-X3.jpg")',
            'url("http://liyu1981.smugmug.com/Photography/365-days-365-photos/i-HwZrx3V/0/X3/IMG_3374-X3.jpg")',
            'url("http://liyu1981.smugmug.com/Photography/365-days-365-photos/i-3mqdZgt/0/X3/IMG_2301-X3.jpg")',
            'url("http://liyu1981.smugmug.com/Photography/365-days-365-photos/i-MpHzMZ7/0/X3/IMG_1448-X3.jpg")'
          ];
          return all[Math.floor((Math.random()*all.length))];
        }
        $('div.aside').css('background-image', geturl());
      });
    </script>
    <div class="aside">
      <div class="avatar circle" style="width: 150px; height: 150px; position: absolute; right: -75px; top: 75px;">
        <div class="center" style="margin-top: 4px; height: 142px; width: 142px; border-radius: 71px; background-image: url('https://secure.gravatar.com/avatar/6a1f8a9d412c8e54bed896135c6f7d0c?s=142')"></div>
      </div>
      <div style="width: 150px; height: auto;">
        <div></div>
      </div>
    </div>
</div>

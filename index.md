---
layout: home
title: "Zaihanoi's Security Lab"
---

# Chào mừng đến với website của tôi!

Tôi là một Security Enthusiast, nơi tôi lưu trữ các tài liệu học tập, Write-up (WU) và các nghiên cứu bảo mật.

### 📂 Danh mục bài viết
<ul style="list-style-type: none; padding-left: 0;">
{% for item in site.wu %}
  <li style="margin-bottom: 8px;">
    <strong><a href="{{ item.url }}">{{ item.title }}</a></strong>
    <span style="color: #666; font-size: 0.9em;"> — {{ item.date | date: "%d/%m/%Y" }}</span>
  </li>
{% endfor %}
</ul>

* **[Blog cá nhân](/blog/)**: Những suy nghĩ và kiến thức tôi tổng hợp được.
---
*Follow me on: [GitHub](https://github.com/zaihanoi)*

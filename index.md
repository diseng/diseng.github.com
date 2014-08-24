---
layout: page
title: diseng's blog
---
{% include JB/setup %}
<table width="100%" rowspan="0" colspan="0">
	<tr>
		<td width="60%">
			<ul class="posts">
			  {% for post in site.posts limit:12 %}
			    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
			  {% endfor %}
			</ul>
		</td>
		<td width="40%" style="vertical-align:top;">
			<div><img alt="diseng" src="{{ ASSET_PATH }}hooligan/img/diseng.jpg"/></div>
			<div>University of Electronic Science and Technology of China</div> 
			<div>Information Security Professional</div>
			<div>Hangzhou , China</div>
		</td>
	</tr>
</table>
<hr>
<div style="width:50%;margin-left:auto;margin-right:auto;text-align:center;clear:both;">
  <a href="/archive.html">view all {{site.posts.size}} articles</a>
</div>



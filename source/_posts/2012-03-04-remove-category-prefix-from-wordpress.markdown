---
layout: post
title: "Remove 'category' Prefix from WordPress"
date: 2012-03-04 13:54
comments: true
categories: php wordpress
---
One of the most pesky things I find about the WordPress installation is that for category pages, WordPress adds the 'category' prefix to URLs. Sure you can live with the behavior (like here on Octopress) but for real-world installs, it looks kind of sloppy and unexpected. I originally came up with this technique in 2007, and after looking at the "new and improved" ways of today, I couldn't find any better options.<!--more-->

Some options I looked at:

* This WordPress <a href="http://wordpress.org/support/topic/how-to-remove-the-category-in-the-link-">support post</a> by <strong>petervcook </strong>says you can change your category base to _./_ and the category prefix will be removed. While this is correct, it will be removed when WordPress generates the links, your category pages won't load. I didn't fully investigate why they weren't loading, but given that post is over four years old and a lot has changed in core since then (including the rewriting mechanism), I don't think it's a good solution.
* Another option I thought of was to handle rewriting of the URLs at the web server layer: internally rewrite your _example.com/sports/_ to _example.com/category/sports/_. This solution would work, but it breaks down when you want to display actual pages (read: this is a page not a category!) and when you look to support custom post types.
* The most popular "solution" is the Top Level Category Plugin, which has now been reworked for WP 3.1+ into the <a href="http://foliovision.com/seo-tools/wordpress/plugins/fv-top-level-categories">FV Top Level Categories plugin</a>. However, this plugin too messes with how WordPress handles rewrites and if any changes occur, this plugin will most likely break. Additionally, one of the comments points out that when this plugin is enabled, custom post types no longer work (fail!).
* Take the <a href="http://techcrunch.com">TechCrunch</a> approach and destroy and notion of category and go full-steam ahead with tags. Sure you can do this, but then you lose the cool category feel! Even TechCrunch is stuck with the _/tag/_ prefix on all their "category" pages.

So what's my answer to this age-old problem? Instead of trying to hack your way through the WordPress rewriting mechanism only to have your code break in some unexpected release when you don't have time to debug, but need one of the new WordPress features right away, use what WordPress already gives you: pages. You see, pages can display post information too! You just need to tell them how. My technique follows a few basic patterns:

* For each category you have, add a page with the same 'slug' as that category
* Write a 'page template' that you'll assign to these pages. The template will handle fetching the posts for the given category. You'll also want some handling for displaying normal pages, which is pretty simple to do.

What benefits does this process have?

* No need to keep track of WordPress rewriting internals. You use a widely supported basic feature of WordPress: **pages**
* Custom post types are supported.

## Show Me the Code
{% codeblock page_type.php lang:php %}
<?php		
	// include this file in your header.php

	if (is_page()) {
		// $is_category_page - true if page is a 'category' page (displays posts), else false
		$is_category_page = false;
		
		// if the page content is empty, it's a category page
		if (empty($post->post_content)) {
			$is_category_page = true;
		}
	}
?>
{% endcodeblock %}

{% codeblock page.php lang:php %}
<?php
	// page.php - use this file as a controller
	include (TEMPLATEPATH .'/header.php');

	// if $is_category_page is true, we know our "page", is really a "category"
	if ($is_category_page)
		// template for your categories
		include (TEMPLATEPATH . '/page-category.php');
	else
		// template for your pages
		include (TEMPLATEPATH . '/page-default.php');

	// etc..
	include (TEMPLATEPATH . '/sidebar.php');
	include (TEMPLATEPATH . '/footer.php');
?>
{% endcodeblock %}
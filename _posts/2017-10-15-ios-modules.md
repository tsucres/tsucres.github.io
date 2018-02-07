---
layout: post
title: 'iOS modules: HeaderedTabScrollView, SwiftyComments & SwiftyMercuryReady'
description: 'Three reusable iOS components written in swift. '
tags: [Swift, iOS, cocoapod, LTS]
date-string: OCTOBER 15, 2017
image:
  feature: 2017-10-15/swift_modules_feature.png
  tinyimg: 2017-10-15/swift_modules_feature_mini.png
---


While developing iOS apps, I often have to make custom components. I like building them independently from the main project. This makes them more maintainable, testable and helps keeping the main project clean. It also makes them easily reusable and implementable in other projects.

Here are three of those components I've built.


### HeaderedTabScrollView


<center>
	<img src="https://github.com/tsucres/HeaderedTabScrollView/raw/master/Screenshots/presentation.gif">
</center>




The main features of this component are: 

- choice between an [ACTabScrollView](https://github.com/azurechen/ACTabScrollView) and [PageMenu](https://github.com/PageMenu/PageMenu)
- fully customizable header view
- support for custom scrolling-animations
- lightweight and easy to use

More info on its [github page](https://github.com/tsucres/HeaderedTabScrollView).

### SwiftyComments

SwiftyComments is highly customizable TableView based component handling hierarchical discussion threads.
<center>
	<img src="https://github.com/tsucres/SwiftyComments/raw/master/Screenshots/ImgurExample.gif">
</center>

The view inside each comment cell are fully customizable. 

Several other examples and an explanation on how it works and how to use it are available on its github repo [here](https://github.com/tsucres/SwiftyComments).

### SwiftyMercuryReady

This module aims to add a "reader" feature to a WKWebView.

<center>
	<img src="https://raw.githubusercontent.com/tsucres/SwiftyMercuryReady/master/Screenshots/darkTheme_small.png">
	<img src="https://raw.githubusercontent.com/tsucres/SwiftyMercuryReady/master/Screenshots/lightTheme_small.png">
</center>

It uses the [Mercury Api](https://mercury.postlight.com/web-parser/) to parse a webpage and retrieve the relevant parts of it (title, body text, etc). Then it loads those informations in a clean HTML template (that's rendered in a WKWebView) which allows to control the font size & style and the theme (dark or light).


The project is made of three parts: 

- a Mercury Api wrapper that just handles the requests to the API;
- a WKWebView subclass rendering the HTML template and the "reader version" of an article;
- a more complete webView allowing the user to enable/disable the reader.


The project is described in more details and available [here](https://github.com/tsucres/SwiftyMercuryReady).


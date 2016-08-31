---
layout: post
title: Fantasy Football Predictions &mdash; A First Look
comments: true
---

A sneak peak at using machine learning to predict fantasy football scores.

<!--more-->

{% include clickToShow.html %}

<script language="javascript"> 
function toggle() {
	var ele = document.getElementById("toggleText");
	var text = document.getElementById("displayText");
	if(ele.style.display == "block") {
    		ele.style.display = "none";
		text.innerHTML = "show";
  	}
	else {
		ele.style.display = "block";
		text.innerHTML = "hide";
	}
} 
</script>
 
# Standard Scoring Predictions
 
<a id="displayText" href="javascript:toggle();">WR Predictions</a>
<div id="toggleText" style="display: none"></div>
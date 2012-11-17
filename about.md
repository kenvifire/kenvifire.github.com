---
layout: page
title: "About"
description: "Kenvi's github blog page"
---
###Kenvi's github blog page
{% include JB/setup %}
<style type="text/css">
   body{
	background:#000;
	color:lime;
	}

	p{
		font-size:14px;
		margin:30px;
		white-space:nowrap;
		overflow:hidden;
		width:30em;
		animation:type 5s steps(50,end) 1;
	}
    @keyframes type{
		from{width:0;}
	}
	span{
		animation:blink 1s infinite;
	}
	@keyframes blink{
		to{opacity: .0;}
	}
</style>

<p>Hello world,this is typing animation powered with CSS<span>|</span></p>
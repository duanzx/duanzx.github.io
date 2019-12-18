---
layout: post
title: java blog title
categories: [java]
---
java blog title1

<div id="mainContentDiv" onmouseover="scrollPlus()" onmouseover="scrollPlus()" style="width:1000px;height:1000px;" ></div>

<script type="text/javascript"> 
 document.getElementById("mainContentDiv").innerHTML = '<object type="text/html" style="width:100%;height:100%" data="/html/2019-12-18-设计模式笔记.html"></object>';

  function scrollPlus(){
  	var h = document.getElementById("mainContentDiv").style.height;
  	var newh = parseInt(h)+1000;
      document.getElementById("mainContentDiv").style.height = newh+"px";
  } 
 
} 
</script>

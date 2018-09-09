---
layout: post
title: "相册"
comments: true     #开启评论功能
date: 2018-01-11 12:46:45
---

<script src="jquery-3.3.1.min.js"></script>
<link rel="stylesheet" href="ins.css">
<div class="photos-btn-wrap">
	<a class="photos-btn active" href="javascript:void(0)">photos</a>
	<a class="photos-btn" target="_blank" href="videos.html">videos</a>
</div>
<div class="instagram itemscope">
	<a href="http://peerywwd5.bkt.clouddn.com/" target="_blank" class="open-ins">图片来自七牛云，正在加载中…</a>
	 <section class="archives album">
        <ul class="img-box-ul"></ul>
    </section>
</div>
<script>
  (function() {
    var loadScript = function(path) {
      var $script = document.createElement('script')
      document.getElementsByTagName('body')[0].appendChild($script)
      $script.setAttribute('src', path)
    }
    setTimeout(function() {
      loadScript('ins.js')
    }, 0)
  })()
</script>
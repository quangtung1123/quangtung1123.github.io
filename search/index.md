---
layout: post
title: Tìm kiếm trong Blog
permalink: /search/

comments: false
tags: 
category: 

---

<!DOCTYPE html>
<html>
<head>
<script language="JavaScript" type="text/javascript" src="/search/lunr.min.js"></script>
<script language="JavaScript" type="text/javascript" src="/search/search.js"></script>
<script language="JavaScript" type="text/javascript" src="/search/jquery.mark.min.js"></script>
</head>
<body>
<div class="container">
    <form action="get" id="site_search">
        <div class="input-group">
            <input id="search_box" type="text" placeholder="type here and enter" />
            <!-- <button type="submit" class="btn btn-default">
                <i class="fa fa-search" aria-hidden="true"></i>
            </button> -->
        </div>
        <ul id="search_results"></ul>
    </form>
</div>
</body>

<style type="text/css">
#site_search {
    width: 100%;
    /*margin: 0 auto;*/
}
#search_box{
    width: 100%;
}
.input-group button{
    width: 100px;
    height: 30px;
    /*background-color: #074B80;*/
}
#search_results {
	/*margin-top: 10px;*/
    width:80%;
}
#search_results p {
    /*margin: 0;*/
    width: 80%;
}

mark, .mark{
    padding: 0;
    /*color: white;*/
    background-color: #FEFE65;
}
</style>
</html>


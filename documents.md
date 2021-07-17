---
layout: article
title: Documents
permalink: /documents/
---

<script>
  (async () => {
    const response = await fetch('https://api.github.com/repos/quangtung1123/quangtung1123.github.io/contents/documents');
    const data = await response.json();
    let htmlString = '<ul>';
    for (let file of data) {
      htmlString += `<li><a href="${file.name}">${file.name}</a></li>`;
    }
    htmlString += '</ul>';
    document.getElementsByTagName('body')[0].innerHTML = htmlString;
  })()
</script>
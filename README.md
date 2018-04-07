# Jekyll Search with Lunr.js
### To get the latest version of Lunr.js visit https://lunrjs.com/

### search.html

```html
---
layout: search
---

<div class="search-page">
	<form class="form-inline my-5" action="search.html" method="get">
		<i class="fas fa-search mr-3" aria-hidden="true"></i>
		<input class="form-control" type="text" id="search-box" placeholder="Search" name="query">
		<input class="btn btn-green btn-sm" type="submit" value="SEARCH">
	</form>

	<ul id="search-results"></ul>
</div>
<script>
	window.store = {
		{% for post in site.posts %}
		"{{ post.url | slugify }}": {
			"title": "{{ post.title | xml_escape }}",
			"author": "{{ post.author | xml_escape }}",
			"category": "{{ post.category | xml_escape }}",
			"content": {{ post.content | strip_html | strip_newlines | jsonify }},
			"url": "{{ post.url | xml_escape }}"
		}
		{% unless forloop.last %},{% endunless %}
		{% endfor %}
	};
</script>

<script src="/assets/js/tools/lunr.js"></script>
<script src="/assets/js/search.js"></script>
```

### search.js

```javascript
(function() {
  function displaySearchResults(results, store) {
    var searchResults = document.getElementById('search-results');

    if (results.length) {
      var appendString = '';

      for (var i = 0; i < results.length; i++) {
        var item = store[results[i].ref];
        // Change tags or add styles below
        appendString += '<li><a href="' + item.url + '"><h3>' + item.title + '</h3></a>';
        appendString += '<p>' + item.content.substring(0, 150) + '...</p></li>';
      }

      searchResults.innerHTML = appendString;
    } else {
      searchResults.innerHTML = '<li>No results found</li>';
    }
  }

  function getQueryVariable(variable) {
    var query = window.location.search.substring(1);
    var vars = query.split('&');

    for (var i = 0; i < vars.length; i++) {
      var pair = vars[i].split('=');

      if (pair[0] === variable) {
        return decodeURIComponent(pair[1].replace(/\+/g, '%20'));
      }
    }
  }

  var searchTerm = getQueryVariable('query');

  if (searchTerm) {
    document.getElementById('search-box').setAttribute("value", searchTerm);

    var idx = lunr(function () {
      this.field('id');
      this.field('title', { boost: 10 }); // title searching has more priority than others
      this.field('author');
      this.field('category');
      this.field('content');
      for (var key in window.store) { 
        this.add({
          'id': key,
          'title': window.store[key].title,
          'author': window.store[key].author,
          'category': window.store[key].category,
          'content': window.store[key].content
        });
      }
    });

var results = idx.search(searchTerm); // Do the search
displaySearchResults(results, window.store);

}
})();
```

### Add the following (not styled) HTML in any page of your project to add a functional search

```html
<form action="search.html" method="get">
<input type="text" id="search-box" placeholder="Search" name="query">
<input type="submit" value="Search">
</form>
```
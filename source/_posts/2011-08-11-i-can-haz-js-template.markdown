---
layout: post
title: ICanHaz templates - Dealing with the lack of dot notation
tags: [javascript, view]
---

The [ICanHaz](http://icanhazjs.com/) javascript templating system is just awesome. It is really simple, and powerful, BUT as all good things in life it's not perfect! ICanHaz is built on top of [moustache.js](https://github.com/janl/mustache.js) that, on his part, is a javascript implementation of the moustache templates. Originally the moustache implementation didn't support dot notation for accessing nested objects. Like for instance

{% codeblock lang:javascript %}
    var post = {
      title : "Post title",
      body : "post body",
      author : {
        name : "Rafael Barbosa",
        age : 26
      } 
    };
{% endcodeblock %}

Normally you can access the author's name in my js files via dot notation like <code>post.author.name</code>, but until [this commit](https://github.com/defunkt/mustache/commit/c183699ff1b23b4bc5efbfa3ed323ff9509855f7) this wasn't possible within moustache's templates and it still isn't supported by the javascript implementation. So how can I access nested objects? The same way you iterate over arrays!

For the post object above, inside an ICanHaz template

{% gist 1161362 %} 

And that'll do it. There's an [open issue](https://github.com/janl/mustache.js/issues/97) on the moustache.js github for the inclusion of the dot notation.



 








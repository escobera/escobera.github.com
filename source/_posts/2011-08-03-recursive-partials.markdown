---
layout: post
title: Show hierarchical data using recursive partials
tags: [ruby, rails, view]
---

Yesterday I needed to show a deep nested tree in a view. The first solution that crossed my mind was something along the lines of a recursive helper like this one.

{% codeblock [language:ruby] %}
def display_tree_recursive(tree, parent_id)
  ret = "<ul>"
  tree.each do |node|
    if node.parent_id == parent_id
      ret += "<li>"
      ret += link_to node.title
      ret += display_tree_recursive(tree, node.id)
      ret += "</li>"
    end
  end
  ret += "</ul>"
end
{% codeblock %}
That's ok for simple ul-li pairs but I needed much more markup since my task was to show this information as nested accordions. Wouldn't it be better if I could define a partial and call it from inside itself passing the children as its `locals` variables? Guess what? It works just like that! =)

In the index view
{% codeblock [language:erb] %}
<ul class="accordion">
    <% @sections.each do |section| %>
      <li>
        <h3><%= section.name %></h3>
        <div class="accordion-content">
          <ul class="accordion">
            <%= render "section/section_children", :section => section %>
          </ul>
        </div>
      </li>
    <% end %>
  </ul>
{% codeblock %}

That `section/section_children` partial is the trick. Inside it we have
{% codeblock [language:erb] %}
<% section.children.each do |child| %>
  <li>
    <h3 ><a href="#" class="header"><%= child.name %></a></h3>
      <div class="accordion-content">
        <ul class="accordion">
          <%= render "section/section_children", :section => child %>
        </ul>
      </div>
  </li>
<% end %>
{% codeblock %}

and that's it!





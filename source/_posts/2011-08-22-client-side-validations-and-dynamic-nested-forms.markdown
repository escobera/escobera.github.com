---
layout: post
title: "Client_side_validations and Dynamic Nested Forms"
date: 2011-08-22 20:59
categories: [rails, view, gem, javascript]
---

Last week I found a little caveat when using the [client_side_validations](https://github.com/bcardarella/client_side_validations) gem with ajax-loaded forms. It didn't work out of the box.

Digging through the sourcecode I found this comment

{% codeblock rails.validations.js %}
  // Main hook
  // If new forms are dynamically introduced into the DOM the .validate() method
  // must be invoked on that form
  $(function () { $('form[data-validate]').validate(); });
{% endcodeblock %}

Ok, so I called <code>$("#my-form").validate()</code> on <code>ajax:success</code> and everything was fine! *Except it wasn't =/*

The behavior I understood, after banging my head against the wall several times was: if the form was entirely rendered in my ajax call it would validate ok, but if I added some fields inside it dynamically then I'd start getting those nasty <code>validators is not defined</code> errors and everything would go boom!

So how do I add some fields to the form and keep them "validatable"?
[this issue](https://github.com/bcardarella/client_side_validations/issues/149) gave me some insight. And after rummaging the code a little more I found that this gem registers the validators in the <code>window</code> object. This process is done inside the form renderer, and that's why the full rendered forms worked and adding just some of the fields did not. To accomplish that I created this helper

{% codeblock lang:ruby %}
  def add_dynamic_validators(builders)
    result = ""
    builders.each do |builder|
      builder.object.client_side_validation_hash.each do |validator|
        result += "window[form_id].validators['#{builder.object_name}[#{validator[0]}]'] = #{validator[1].to_json};\n"
      end
    end
    result.html_safe
  end
{% endcodeblock %}

It recieves an array of form builders and inspect it for the validators added by the gem. Then it prints the resulting instruction within the form partial, inside a function, to which I pass the id of the form that I want to validate.

{% codeblock lang:javascript %}
  function update_dynamic_fields(form_id)
  {
    <%= add_dynamic_validators builders %>
  }
{% endcodeblock %}

Then after the fields (with the script block) were added the to form you can call

{% codeblock lang:javascript %}
  var $form = $("#my-form");
  update_dynamic_fields($form.attr("id"));
{% endcodeblock %}

I know this is hacky as hell, but with this I was able to keep using this gem even inside pages with lots of moving parts.



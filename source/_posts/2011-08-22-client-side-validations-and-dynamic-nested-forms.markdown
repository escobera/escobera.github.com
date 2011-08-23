---
layout: post
title: "Client_side_validations and Dynamic Nested Forms"
date: 2011-08-22 20:59
categories: [rails, view, gem, javascript]
published: false
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

[[ Inserir o helper aqui ]]

It recieves an array of form builders and inspect it for the validators added by the gem. Then it prints the resulting instruction within the form partial, inside a function, to which I pass the id of the form that I want to validate.

[[ Inserir script tag aqui ]]

I know this is hacky as hell, but with this I was able to keep using this gem even inside pages with lots of moving parts.



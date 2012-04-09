---
layout: post
title: "Nginx + Upload Module and Carrierwave"
date: 2012-04-09 11:15
comments: true
published: false
categories: [rails, nginx, carrierwave]
---

Last night was a really long one. Why? Because I couldn't find enough information on how to make nginx handle uploads (using upload module and upload progress) and include carrierwave on the equation to handle files after they have been sent to the server.

Maybe I didn't search enough, maybe I found only the old deprecated stuff, but after this(link) three(link) sources(link) I think I came up with a decent solution.

First off, let's compile nginx with some upload sugar on top.

(add instructions to compile nginx with upload module and upload progress)

And the `nginx.conf` to handle that. Restart nginx and you're good to go. #NOT

As soon as you try to upload anything nginx will complain about permissions or directories it could not found so lets fix that.

(cmd to create the directories and chown to nginx user)

Now back to your app.

{% codeblock lang:ruby %}
def upload
  @track = @album.tracks.build(params[:track])

  # Production/staging only, development will go the carrierwave way.
  if Rails.env != "development"
    @track.raw_file = ActionDispatch::Http::UploadedFile.new(
      filename: params['track']['raw_file']['original_filename'],
      tempfile: File.open(params['track']['raw_file']['path'])
    )
    @track.name = params['track']['raw_file']['original_filename'].chomp(".wav")
  else
    @track.name = params['track']['raw_file'].original_filename.chomp(".wav")
  end

  @track.save!
  render :json => @track
end
{% endcodeblock %}

The trick here is simulate a ActionDispatch::Http::UploadedFile being passed to carrierwave.

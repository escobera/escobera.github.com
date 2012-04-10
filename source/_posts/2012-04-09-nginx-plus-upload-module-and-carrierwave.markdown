---
layout: post
title: "Nginx + Upload Module and Carrierwave"
date: 2012-04-09 11:15
comments: true
published: true
categories: [rails, nginx, carrierwave]
---

Last night was a really long one. Why? Because I couldn't find enough information on how to make nginx handle uploads (using upload module and upload progress) and include carrierwave on the equation to handle files after they have been sent to the server.

Maybe I didn't search enough, maybe I found only the old deprecated stuff, but after compiling [these](http://fernando.blat.es/post/11106552363/nginx-upload-module-rails-carrierwave) [sources](http://blog.joshsoftware.com/2010/10/20/uploading-multiple-files-with-nginx-upload-module-and-upload-progress-bar/) I think I came up with a decent solution.

First off, let's compile nginx with some upload sugar.

{% codeblock lang:bash %}
cd /tmp

wget http://nginx.org/download/nginx-1.1.14.tar.gz
tar -zxvf nginx-1.0.14.tar.gz

wget http://www.grid.net.ru/nginx/download/nginx_upload_module-2.2.0.tar.gz
tar -zxvf nginx_upload_module-2.2.0.tar.gz

cd nginx-1.0.14
./configure --prefix=/usr/local/nginx-1.1.14 --with-http_ssl_module --with-http_gzip_static_module --conf-path=/etc/nginx/nginx.conf --add-module=../nginx_upload_module-2.2.0/
make
sudo make install
{% endcodeblock %}

So the module is there, we need to make it come to life in `nginx.conf`

{% codeblock lang:nginx %}
http {
  # ... A lot of stuff
  server {

    # This will be our route on our rails app
    location /videos/upload {
      # app_server is the unicorn upstream
      proxy_pass http://app_server;

      # @app is the unicorn upstream proxy location
      upload_pass @app;

      upload_store /web_apps/mysite/shared/uploads/tmp 1;

       # set permissions on the uploaded files
      upload_store_access user:rw group:rw all:r;

      # Set specified fields in request body
      upload_set_form_field $upload_field_name[original_filename] "$upload_file_name";
      upload_set_form_field $upload_field_name[content_type] "$upload_content_type";
      upload_set_form_field $upload_field_name[path] "$upload_tmp_path";

      upload_pass_form_field "^X-Progress-ID$|^authenticity_token$";
      upload_cleanup 400 404 499 500-505;
    }
}
{% endcodeblock %}

This is from a server using unicorn to serve the rails app, if you need some help getting started with nginx and unicorn [this](http://tomkersten.com/articles/nginx-unicorn-rvm-server-setup/) is a good place to start. And I really recommend that you read [upload module's docs](http://www.grid.net.ru/nginx/upload.en.html) carefully.

The important bits are the ones starting with "upload".
The way it works is `upload_pass` routes the request to the `@app` location after the upload has finished it'll pass the form fields you set with `upload_pass_form_field` along with the ones we'll be using to show the file to carrierwave.

Now restart nginx and you're good to go. #NOT

As soon as you try to upload anything nginx will complain about permissions or directories it could not found so lets fix that.

I'm using 1 level deep directory hashing (set on `upload store`) so I'll have to manually create the directory structure and chown it to the user that runs nginx workers (in my case is rafa:rafa).

{% codeblock %}
cd /web_apps/mysite/shared/uploads/tmp
mkdir 0 1 2 3 4 5 6 7 8 9
sudo chown -R rafa:rafa /web_apps/mysite/shared/uploads/tmp
{% endcodeblock %}

Now back to your rails app. Lets create the route to recieve the request after the upload is handled by nginx.

{% codeblock routes.rb %}
  match "/videos/upload/", as: "upload_videos", controller: "videos", action: "upload", via: :post
{% endcodeblock %}

And the action to save it.

{% codeblock videos_controller.rb %}
def upload
  @video = Video.build(params[:video])

  # Production/staging only, development will go the carrierwave way.
  if Rails.env != "development"
    @video.raw_file = ActionDispatch::Http::UploadedFile.new(
      filename: params['video']['raw_file']['original_filename'],
      tempfile: File.open(params['video']['raw_file']['path'])
    )
    @video.name = params['video']['raw_file']['original_filename'].chomp(".raw")
  else
    @video.name = params['video']['raw_file'].original_filename.chomp(".raw")
  end

  @video.save!
  render :json => @video
end
{% endcodeblock %}

The trick here is to simulate a `ActionDispatch::Http::UploadedFile` being passed to carrierwave.

This will send the file to the right place, but I still have some quirks to solve. Like the temporary dir cleanup task and the content_type validation. Once I get the hang of those I'll update this post.

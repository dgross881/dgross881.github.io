---
layout: post
title: "Advanced feature specs" 
date: 2014-04-22 14:40
comments: true
categories: [feature specs]
---

  Writing Advance Features for queue items index page. Haml makes it very easy to
assign a dom_id to specific html tag. In my example below I used it on %tr[queue_item.video]. 
This allows to the ability to identify a specific table with the video_id format. If you view your
code you will now have the id='video_#' within the tr tags.  

[^1]:[dom_id](http://api.rubyonrails.org/classes/ActionView/RecordIdentifier.html).
~~~

  - @queue_items.each do |queue_item| 
   %tr[queue_item.video]
     = hidden_field_tag "queue_items[][id]", queue_item.id 
     %td=text_field_tag "queue_items[][position]", queue_item.position 
     %td
       = link_to queue_item.video_title, queue_item.video
     %td
~~~
{:lang="ruby"}

  Dom_id helper method located at spec/support/dom_id_for_helper.rb. The # sign is used as a shortcut to
specify the id attribute of an element. ActionView::RecordIdentifier.dom_id will generate 
queue_item.video.id in the form of "video_id". Example: video_1, video_2 , etc..

~~~

module RailsDomIdHelper
  def dom_id_for(video) 
    ['#', ActionView::RecordIdentifier.dom_id(video)].join
  end 
end 

~~~
{:lang="ruby"}

   In this example dom_id allows me to easily test the Rspec features of my
 queue_items ability to change positions in the queue. I am using the support method dom_id_for 
 in a within capybara block on the set_video_position method.

~~~

feature "Users queue items" do 
  scenario "Signed in user can add queue_items" do 
    comedy = Fabricate(:category)
    southpark = Fabricate(:video, title: "southpark", category: comedy) 
    futurama = Fabricate(:video, title: "futurama", category: comedy) 
    familyguy = Fabricate(:video, title: "family guy", category: comedy)
    user = Fabricate(:user)
    sign_in user  
    
    add_video_to_queue(futurama)
    add_video_to_queue(familyguy)
    set_video_position(familyguy, 2)
    set_video_position(southpark, 3)
    set_video_position(futurama, 1)
    
    click_button "Update Instant Queue"
    
    expect(user.queue_items.first.video_id).to eq(futurama.id) 
    expect(user.queue_items.second.video_id).to eq(familyguy.id)
    expect(user.queue_items.third.video_id).to eq(southpark.id)
  end 
end 
  
  def set_video_position(video, position)
    within dom_id_for video do
      fill_in "queue_items[][position]", with: position 
    end 
  end 
  
  def add_video_to_queue(video)
    visit home_path 
    find("a[href='/videos/#{video.id}']").click
    click_link "+ My Queues" 
  end 
end

~~~
{:lang="ruby"}










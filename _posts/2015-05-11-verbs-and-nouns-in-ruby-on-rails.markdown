---
layout: post
title:  "Verbs and Nouns in Ruby On Rails"
date:   2015-05-11 12:40:52 +0200
categories: REST
---

# The Motivation

I've heard a lot about REST in my early days of Rails (Rails 3 was just around the corner, so it's not THAT early) and I understood it was using the HTTP verbs GET, PUT (now PATCH), POST and DELETE and allowed Rails developers to have statelessness in their URIs, among other things.

I read up on it and understood that REST allowed for proxies and caching. I was happy with that, went along with it and created routes like this:
<pre>resource :reports, only: :show do
  member do
    get :line_item_overview
    get :invoices_us
    get :invoices_uk
    get :fees
    get :cc_payments
    get :sales_by_product
    get :donations
    ...
  end
end</pre>

There's more, but you get the point. Also see <a href="https://www.youtube.com/watch?v=Gt0M_OHKhQE#t=1675" target="_blank">Robert Martin's opinion on reports</a>.

I understood the <code>resource</code> keyword as a nice shortcut to get the basic CRUD actions for my model. Whenever I needed "something special", I created a <code>member</code> or a <code>collection</code> route. This seemed like the Rails way to me.

I just ignored the warning given right above <a title="non-resourceful-routes in the rails routing guide" href="http://guides.rubyonrails.org/routing.html#non-resourceful-routes" target="_blank">non-resourceful-routes</a> in the rails routing guide:
<blockquote>If you find yourself adding many extra actions to a resourceful route, it's time to stop and ask yourself whether you're disguising the presence of another resource.</blockquote>
Yeah whatever. What's too many anyway? I didn't even start to think that there could be some theory behind resource orientation. But now I want to know what it would look like to have my example be resource oriented.
<h1>The research</h1>
I surfed around a bit and found an explanation of Resource Oriented Architecture (ROA):
<blockquote>ROA is just four concepts:
<p style="padding-left: 30px;">1. Resources</p>
<p style="padding-left: 30px;">2. Their names (URIs)</p>
<p style="padding-left: 30px;">3. Their representations</p>
<p style="padding-left: 30px;">4. The links between them</p>
and four properties:
<p style="padding-left: 30px;">1. Addressability</p>
<p style="padding-left: 30px;">2. Statelessness</p>
<p style="padding-left: 30px;">3. Connectedness</p>
<p style="padding-left: 30px;">4. A uniform interface</p>
</blockquote>
"Just", without any explanation. It's a bit like saying set theory is just 10 axioms:
<ol>
    <li><span class="toctext">Extensionality</span></li>
    <li><span class="toctext">Regularity</span></li>
    <li><span class="toctext">Schema</span><span class="toctext"> of </span><span class="toctext">specification</span></li>
    <li><span class="toctext">Pairing</span></li>
    <li><span class="toctext">Union</span></li>
    <li><span class="toctext">Replacement</span></li>
    <li><span class="toctext">Infinity</span></li>
    <li><span class="toctext">Power</span><span class="toctext"> set</span></li>
    <li><span class="toctext">Well-ordering</span></li>
    <li>Choice (to have ZFC)</li>
</ol>
Now go and proof that the <a title="Continuum hypothesis" href="http://en.wikipedia.org/wiki/Continuum_hypothesis">Continuum hypothesis</a> is independent of ZFC...

Turns out the source for these 4 concepts and 4 properties is <a title="Restful Web Services - All block quotes in this article have this as their source" href="http://www.crummy.com/writing/RESTful-Web-Services/RESTful_Web_Services.pdf" target="_blank">this little gem</a>. Assuming you didn't read it before finishing this paragraph I'll try to sum up what seems important to me here:
<blockquote>Resource-Oriented Architecture is also RESTful. But REST is not an architecture: it’s a set of design criteria. You can say that one architecture meets those criteria better than another, but there is no one “REST architecture.”

...

In RESTful architectures, the method information goes into the HTTP method. In Resource-Oriented Architectures, the scoping information goes into the URI. The combination is powerful.</blockquote>
Ok. That's all pretty railsy. Notice that query parameters are part of the URI, so it's uncommon to see a rails app that has scoping information somewhere else. I guess scoping for the current_user without including them in the url might be the most common violation of this principle.
<h2>The "four" concepts</h2>
<blockquote>A <strong>resource</strong> is anything that’s important enough to be referenced as a thing in itself.</blockquote>
So in the routes file above I can find a couple of resources:
<ul>
    <li>the fees-report</li>
    <li>the report for us-invoices</li>
    <li>the report for uk-invoices</li>
    <li>...</li>
</ul>
I didn't use the <code>resource</code> keyword in my example, though. Maybe I should? Let's read on:

A <strong>name </strong>of a resource is their URI. In my example <code>my-site.org/reports/invoices_us</code> is a <strong>name </strong>for a resource. Note that <code>my-site.org/reports/invoices?country="us"</code> would be another name for the resource, as parameters are a part of the URI (I wrote that a second time, because I keep on looking that up).

A <strong>representation </strong>of that resource would be the HTML for that report. Another one might be a PDF, or even a JSON-String with the data, or even the printed report on paper. The resource can have multiple names in that case for different representations of it:
<blockquote>A web server can’t send an idea; it has to send a series of bytes, in a specific file format, in a specific language. This is a <strong>representation</strong> of the resource.</blockquote>
The main point seems to be that the resource is independent as a concept from its representation and it's name. You never really <em>get</em> a resource, only their representations. I think that might be helpful somehow...

I honestly don't really get why <strong>links </strong>are in the list as well. Can't a JSON-API without links to other resources still resource oriented? I'll just ignore it for this post and I'll also ignore the <strong>connectedness</strong> property because the article is "calling the quality of having links connectedness<strong>."</strong>
<h2>The "four" properties</h2>
<blockquote>Since resources are exposed through URIs, an <strong>addressable</strong> application exposes a URI for every piece of information it might conceivably serve. This is usually an infinite number of URIs.</blockquote>
Seems obvious but again, having information scoped by a current user, that's not set from the current_url means that not all pieces of information are addressable. Also In my example above there is not an infinite number of URIs. Query parameters are the reason for the infinite number and I don't have them for my reports.
<blockquote><strong>Statelessness</strong> says that the possible states of the server are also resources, and should be given their own URIs.</blockquote>
A state can be a search result, or the result of any other algorithm, so that seems unsurprising to me as well. I get the feeling this deserves more attention, but I have to get to the end.
<blockquote>The important thing about REST is not that you use the specific <strong>uniform interface</strong> that HTTP defines. REST specifies a uniform interface, but it doesn’t say which uniform interface. GET, PUT, and the rest are not a perfect interface for all time. What’s important is the uniformity: that every service use HTTP’s interface the same way. The point is not that GET is the best name for a read operation, but that GET means “read” across the Web, no matter which resource you’re using it on.</blockquote>
Why is that good? You have THE SAME verbs in all the internet! It's like getting vaccinated: It helps others. The variance will only be in the resources and their names. This is less useful if you click around on a website. But it's really nice for APIs. Theoretically you could write one library that just have to know the resource names of each consumed service, if everyone would adhere to ROA. Theoretically...
<h1>A better version</h1>
OK. So all this is not a big deal for my example. I had a resource oriented architecture before. I just didn't set it up within the Rails conventions. Understanding what a resource is lead me to rewrite my routes to look like this:
<pre>namespace :reports do
  resource :line_item_overview, only: :show
  resource :invoices, only: :show
  resource :fees, only: :show
  resource :cc_payments, only: :show
  resource :sales_by_product, only: :show
  resource :donations, only: :show
  ...
end</pre>
Which has the added bonus of me being nudged to move logic out of my mighty reports controller.
<h1>Conclusion</h1>
Even if my example didn't change much I learned a lot. I keep on underestimating how much theory is behind rails and the web in general. There is a lot to know about software architecture. For example <a href="http://research.microsoft.com/pubs/117710/3-arch-styles.pdf" target="_blank">this presentation</a> contrasts
<ul>
    <li>object orientation (as in ruby!)</li>
    <li>resource orientation (which I just described)</li>
    <li>service orientation (as in SOA everyone is talking about)</li>
</ul>
that just blew my mind.

I've spent way more time on this article then I planned, but I still managed to leave out the the main difference between REST and Remote Procedure Calls (RPC). The difference is that the "method information" goes into the HTTP-verb. But I'm still unsure if that's REST or ROA and I'm unsure what exactly "method information" is. I think it should be the difference between GET <code>my-site.org/counter?reset</code> and POST <code>my-site.org/counter-reset</code> but I don't have the time to make sure right now. I'll just try not to write my own verbs and instead have a counter-reset resource instead of a <code>CounterController</code> with a reset action for now and research a bit more later.]]></content:encoded>

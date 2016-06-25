---
layout: post
title:  "Specific Absence in Ruby On Rails"
date:   2015-01-01 12:40:52 +0200
categories: NullObject
---


I recently stumbled upon this pattern in <a title="Confident Ruby" href="http://www.confidentruby.com/" target="_blank">Avdi Grimm's book Confident Ruby</a> and have been fascinated with it ever since. I keep on thinking about how long it took for Europeans to include the concept of the number 0 in their arithmetic.
The concept of absence or emptiness is quite easy to implement in object-oriented languages, but it's not intuitive. We tend to think about what's there, not about absence. If the concept of a <code>NullObject</code> is at all similar to 0 in math (and I think it is), it feels like there should be quite some potential for easier understanding and consistency, like in math when 0 became just another number.

One might argue that most languages have the concept of <code>Null</code> or <code>nil</code>, but a "general absence" is a lot less helpful for our mental models than a specific one. Imagine emptiness and/or absence. How many calories does it have? What's its serial-number? Show me a list of its documents. These questions seem strange. Had I asked you how to imagine no food, no thing in a store, no folder the questions would be a little less nonsensical. It makes sense to answer specific questions for specific <code>NullObjects</code> while it is impossible to imagine all possible questions and answer them, because we expect a different kind of answer (0, '', []) or behaviours of emptiness, depending on the context.
<h1>The pattern</h1>
Suppose we created our own payment processing system that includes the following:
<pre>class PaymentProcessor
  def process_new_payments(payments_to_be_processed)
    payments_to_be_processed.each do |payment|
      process_payment(payment)
    end
  end 

  def process_payment(payment)
    Order.find(payment.order_id).mark_as_paid_by(payment)
  end
end</pre>
At the first look this seems to do the trick and is easy to understand.
What if the data is faulty though? <code>Order.find</code> will raise an exception, if the <code>order_id</code> is not in you system and crash your whole payment processing. You probably want to rescue exceptions that happen during <code>process_payment</code> anyway, but that's not the point of this post.

In the past I would've done something like this to prevent the exception from happening:
<pre>def process_payment(payment)
  order = Order.find_by(id: payment.order_id)
  if order.present?
    order.mark_as_paid_by(payment)
  else
    payment.mark_as_order_missing
  end
end</pre>
<code>process_payment</code> is harder to understand. There is an <code>if...else-block</code> now. Reading an <code>if...else-block</code> like this feels to me like being invited to dinner asking what the food will be and instead of answering "Pizza!", they answer: "Well, you know, we could get some Sushi, or maybe some Chinese food? Hmm." If you're anything like me that's a little unsatisfactory. While they do come up with an answer eventually, I didn't really wanted to know what the options are, I just wanted an answer (and I wanted it to be Pizza, but that's not the point here...).

There is a principle that's called "tell don't ask" that's violated here. We don't just tell the order to do something. We're asking it something first. To make things worse we have to ask the order, if it's there at all (<code>order.present?</code>). It's like your friend getting all philosophical about if the world is just an illusion anyway, when all you wanted is pizza. That's called a nil-check. The method doesn't know, if there is an order or not as <code>find_by</code> returns <code>nil</code> when it doesn't find what it was looking for. So it has to ask it, if it's really there.
With a <code>NullObject</code>:
<pre>def process_payment(payment)
  order = Order.find_by(id: payment.order_id) || NullOrder
  order.mark_as_paid_by(payment)
  payment.mark_as_order_missing if order.is_a?(NullOrder)
end</pre>
And somewhere:
<pre>class NullOrder
  def mark_as_paid_by(payment)
  end
end</pre>
Now the method knows there is an order. You can confidently tell it to mark the order instead of asking if it's really there.

There still is an <code>if</code> I sneakily hid in the last line of the method:
<pre>def process_payment(payment)
  order = Order.find_by(id: payment.order_id) || NullOrder
  order.mark_as_paid_by(payment)
end</pre>
And somewhere:
<pre>class NullOrder
  def mark_as_paid_by(payment)
    payment.mark_as_order_missing
  end
end</pre>
Finally pizza. I think it's a lot easier to understand, because it keeps me from thinking about conditions. I just have to properly define how this special kind of emptiness should behave.

I'm suspecting someone may find it problematic that a <code>NullObject</code> does something though. They might expect only empty-methods. Also it might be unexpected to have the payment changed in <code>mark_as_paid_by</code>. These are probably not the only pitfalls for my <code>NullOrder</code>. One inherent gotcha of this pattern is having nil-checks on a <code>NullObject</code> somewhere in the code now. How should a <code>NullOrder</code> react, if it ends up in some view, for example?
I feel like these problems are a little bit like dividing by zero. Weird edge cases that might need some workaround, but I'd say it's worth it.

What do you think?

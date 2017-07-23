---
layout: post
title:  "Do one thing well in Ruby On Rails"
date:   2015-02-03 12:40:52 +0200
categories: Single-Responsibility-Principle
---

<h2>My Motivation</h2>
During my years with Rails I've come across the term Single Responsibility Principle (<em>SRP</em> for short) a lot. I think someone explained it to me something like this: "A class should do only one thing." or was it a method? What is this "one thing" aka a responsibility? With my naive understanding of what a responsibility is, everything violates <em>SRP</em>. It seems about perspective only. I mean, if something's reading and writing a variable, it's responsible for two things, right?

I've frequently heard someone say that ActiveRecord kind of violates <em>SRP</em>, because persistence is mixed with other logic. So I asked myself, if ActiveRecord <strong>as a pattern</strong> violates <em>SRP</em> and found <a href="http://programmers.stackexchange.com/questions/228672/doesn-t-active-record-violate-srp-and-ocp" target="_blank">this</a>. To sum that up for you: "no.". So the problem everyone is talking about is putting too much logic in an ActiveRecord model (like all those other Rails Apps I've seen... and written) and to make that clear: <strong>Any</strong> kind of logic that's not about manipulating the database is considered too much from the perspective of <em>SRP</em>.
<h2>My Example</h2>
I'm working on an app right now that has some intensive calculations going on and I have some calculator models that combine different results of calculations done in the database. From my understanding executing these calculations should be a responsibility. But here comes the problem: Some calculator classes take a lot of different options, to allow different variants of the calculation. Which one might say are technically two different calculations. Are these two responsibilities then? Would I have to write a new class for every calculation not to violate <em>SRP</em>? I've recently spent a week feeling smart about refactoring a couple of these calculator classes that can combine and calculate the heck out of our data. Here's kind of how it looks:
<pre>class EnergyConsumptionCalculator
  def initialize(consumption_container)
    @db_results = EnergyConsumptionCalculatorFetcher.new(saving_container)
  end

  def electricity
    set_consumption_scope(:electricity)
    self
  end

  def combustibles
    set_consumption_scope(:combustibles)
    self
  end

  def for_years(years)
    set_years(years)
    self
  end

  def kWh(options)
    result = if options.include? :heating_adjusted
      heating_adjusted_kwh
    elsif options.include? :weighted
      weighted_kwh
    end

    result -= exports_kwh if options.include? :exports_deducted
    result
  end

  private

  def heating_adjusted_kwh
    ...
  end

  def weighted_kwh
    ...
  end

  def exports_kwh
    ...
  end
end
</pre>
Where <code>heating_adjusted_kwh</code>, <code>weighted_kwh</code> and <code>exports_kwh</code> choose the right kind of fetched data based on the set scope and years.
<h2>The verdict</h2>
Does my calculator hold up against the haunting eye of <em>SRP?</em> The one responsibility of this calculator is to compute the energy consumption of the facility. So yes, ... maybe? I mean even fetching the basis for these calculations is in another class. This is so much more separation than what I've done before. These calculations are independent of persistence. Isn't this really well separated? I kind of feel like it, but I just don't know. My understanding of what a responsibility is is not grounded in anything but hearsay and my understanding of the word responsibility in everyday life. I'm unsure about my calculator, so I'll go and research a bit.

Wikipedia says a characterization of <em>SRP</em>-compliance is having exactly <strong>one reason to change</strong> and that helps a little. So my calculator would need to change, if the logic for calculating kwh changes. Sounds good to me. On the other hand my calculator would have to change, if the heating adjustment logic changes OR if exports needs to be weighted OR if I wanted to have another option for the calculation OR if there is another kind of consumption_scope. Now it looks looks grim for my calculator. Hmm... Let's look into what the rest of the internet has to say about <em>SRP</em>.

[2 hours later] Oh no... I thought I was pretty smart with views and controllers that don't have a lot of logic. On our recent project I felt even smarter when we started moving logic out of our <em>AR</em>-models into what we called "service objects". Now I read up on <em>SRP</em> and I think we could've been SOOO much more systematic with it. I just want to start over... or at least go and refactor everything. Damn.
<h2>The verdict - again</h2>
OK. So, what did I learn?

I read <a href="http://code.tutsplus.com/tutorials/solid-part-1-the-single-responsibility-principle--net-36074" target="_blank">this</a> and I think I understood <em>SRP</em> a lot better. I also think that the world doesn't need another post about <em>SRP</em>. But my mission is have a look at those patterns applied in Rails, right? So, let's see if anyone has done that already. <a href="http://naildrivin5.com/blog/2012/06/10/single-responsibility-principle-and-rails.html" target="_blank">Oh boy</a>, now I really think I don't have anything to add to the discussion. Except for digging this up again and maybe to add my fabulous calculator example to the mix. So far I completely agree with Mr. Copeland here. Which also means that almost all models and controllers I wrote are doing too much.

But one simple guideline intrigued me the most: "An SRP’ed class is reflected by having <strong>exactly one public method</strong>." (<a href="http://nicksda.apotomo.de/2014/05/rails-misapprehensions-single-responsibility-principle/" target="_blank">supposedly said by Uncle Bob Martin at a Lonestar keynote</a>). I didn't hear him say it, but I found <a title="Robert Martin on SRP" href="https://www.youtube.com/watch?v=Gt0M_OHKhQE" target="_blank">a talk by him about SRP</a>. Let's just assume he did propose only one public method. Now this is new to me. O_O I'm flabbergasted... but seriously intrigued! This feels right somehow. One class does one thing, one public method. This is definitely NOT what I was doing. I mean I tried to keep their number small, but I never even thought of having only one.

So let's have a look at my calculator:

I have 5 public methods. That's too much. Arguably only <code>kwh</code> is really doing something. The rest is just configuration and I just really like to read stuff like <code>calculator.electricity.for_years([2013, 2014]).kwh(:heating_adjusted)</code>.
So, if we ignore initialize I might be able to convince someone <em>SRP</em> is not violated here, I guess, but after reading up on it, it doesn't feel right anymore, especially because heating adjustments and weighting is done in the same class. But then again after listening to Bob Martins talk about <em>SRP</em> I'd say the same person/role would demand a change in the calculator. It's always the committee in charge of the calculation algorithms.

Ok, so I did not present an application of this pattern like I wanted to. I'm not even sure yet, if my example is compatible with <em>SRP</em>. I didn't leave any thoughts on <em>SRP</em> applied to Rails Controllers and ActiveRecord Models. Now I'm tired and the post is getting too long. I'm still glad I wrote this, because of everything I learned from it though. Maybe you have something to say about it?

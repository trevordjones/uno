---
title: 'What is a Queue and how to Implement One'
date: '2019-08-12'
categories: [Data Structures]
tags: [queue, data structures]
---

The Queue data structure is actually pretty simple. It can get more complex with something called a Primary Queue, but let's just start with a Queue. If you think of any line of people waiting it primarily works as first in first out, or FIFO. If you're the first in line, then you're the first one to get service and the first to leave the line. In non-computer sciencey terms, that's what a queue is. Same goes for a Queue in computer sciencey terms.

More formally in computer science terms, a Qeueu also supports to main operations:
1. push_back(e)
2. pop_front()
`push_back` add `e` to the back of the queue. `pop_front` returns the first element in the Queue and "pops" (removes) it from the continue.

Sometimes data needs to get queued up to wait and be processed at some future time. A good example of this is from a previous job of mine. We worked with customer's videos and they uploaded their videos to our service. They uploaded an mp4 file and we transcoded them to m3u8 renditions so they could be streamed. This takes time and we only had so many servers to do it (each server could only do one video at a time). So if a single customer or multiple customers uploaded hundreds of videos at once, we would have to queue them up. When a video became the first in line it went to a server to be transcoded as soon as one was available. This is normally what is happening with any app that is processing multiple of any kind of data (images, video, grading tests, etc).

Now you might be wondering if programming languages support a Queue like they do String, Integer, Array, Hash (Dict), etc. Those are all data structures as well. Many popular ones do not (Ruby, Python, JavaScript, etc), but I do know that Erlang does. So my guess is there are others that do. But if the language we are using does not have a Queue built-in, how do we use one? We make it!

### Implementing a Queue
If all you want to know is what a Queue is, then you're done with the post. If you want to make and use a Queue, then read on.

Let's make a Queue using Ruby and then using Elixir. That way we can see an Object Oriented approach and then a functional approach.

### Ruby Queue
This is actually quite trivial. Something like this will suffice:

{% highlight ruby linenos%}
  class Queue
  class DataTypeMismatch < StandardError
  end

  attr_reader :data_type
  attr_accessor :elements

  def initialize(data_type)
    @data_type = data_type
    @elements = []
  end

  # let's add those two necessary methods

  def push_back(e)
    # we'll make sure all elements in our queue have the same data type
    unless e.is_a?(data_type)
      raise DataTypeMismatch, 'Queue must contain the same data type'
    end

    @elements << e
  end

  def pop_front
    # Ruby's pop method actually returns the last element.
    # You could also rename this method to be shift_front to better correlate
    elements.shift
  end
end
{% endhighlight %}

One thing to note here is that we have a queue for a specific data type. This is not necessarily to making a queue, but might make life easier if you know what data types you're processing. It would be bad to write code to process Strings from a Queue, but you inserted an Array by mistake.

Now let's use it:

{% highlight ruby linenos %}
begin
  queue = Queue.new(String)
  queue.push_back("Is this a queue?")
  puts queue.elements
  queue.push_back("I believe it is")
  puts queue.elements
  puts queue.pop_front
  # Now it contains only the second argument
  puts queue.elements

  # What if we try to add an array?
  queue.push_back(["hello"])
rescue Queue::DataTypeMismatch => e
  puts e.message
end
{% endhighlight %}

## Elixir Queue
Now for some functional practice. Remember that with Functional programming, we do not have any class objects or ways to store state (like with the instance variables). We only have functions that take an input, run some logic on it, and deliver the output. With pure functions, it should be able to take the same input and produce the same output every time.

While Elixir doesn't have classes, it does have modules which are useful for grouping functions that are alike. So we'll create a module for our Queue.

This code does come with a warning however that Elixir is not the best option for making a Queue.

{% highlight elixir linenos %}
defmodule Queue do
  def push_back(queue, e), do: queue ++ [e]

  def pop_front([]), do: [nil, []]
  def pop_front([head | tail]), do: [head, tail]
end
{% endhighlight %}

This is ok for smallish queues, but as they grow to the tens of thousands, we shouldn't use Elixir Lists. Lists in Elixir are Linked Lists, so by nature, adding to the end of a List is an expensive operation. So as your Queue grows it will get slower as more is added. Erlang, however, has Queue functionality built right in that we can use. Since Elixir is run on the Erlang VM, we can write Erlang right in our Elixir app.

{% highlight elixir linenos %}
{% raw %}
queue = :queue.new
queue = :queue.in(1, queue)
queue = :queue.in(2, queue)
queue
# > {[2], [1]}
{{:value, head}, queue} = :queue.out(queue)
head
# > 1
queue
# > {[], [2]}
{% endraw %}
{% endhighlight %}

So in practicality, you should just use the `:queue` that's built into Erlang instead of building your own.

That's all for Queues!

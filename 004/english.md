In this week's episode we're going to decorate the calendar week object that we created last week so that it doesn't just return a matrix of days. It returns the days along with some CSS information that we can then use to style it. Then, we're going to plug this little library into a Rails app and create some views and partials to render it out.

Here's the file we created last week. We're going to start by creating a new object. We're going to call it `Calendar.` It's going to wrap the other object, so for more or less, that just means that we're going to use the same interface. Let me save off this instance variable so we have access to it and all of our other methods. We're going to have one method here publicly called `to_a,` and this method is going to return the previous object `CalendarWeeks.new.` Pass in the date, `.to_a` ... What we've done at the moment is just rename our object from `CalendarWeeks` to `Calendar.` If we rerun our file here, we can just replace `Calendar` for `CalendarWeeks,` and we get the exact same behavior we got before out of the CalendarWeeks object.

~~~ruby
class Calendar
  def initialize(date=Date.today)
    @date = date
  end

  def to_a
    CalendarWeeks.new(@date).to_a
  end
end
~~~

The next step is going to be to decorate this information with the CSS. We're going to do that by mapping over the return of the CalendarWeeks object. Each iteration on this outside is going to give us a week, so then if we map over the week we get dates. Again, if we were to stop right here, we haven't actually changed the behavior of the application. Same behavior. 

![Calendario](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/004/images/001.png)

Now, let's change that behavior. Instead of just returning the date ... If we returned the date as an initial value, and as a second value we returned some CSS classes for that date, we could then make a private method called `css_classes_for(date).` Inside of there, we would return something that looked more or less like `past` and `other_month` because ... Let's say we start with the first calendar date as last month. It's in the past. It's also not part of this calendar month. It's part of last calendar month. Then, we would want to compact that calendar list and to get rid of any possible null values. Then, we join it with a space, and what we end up getting is a value that looks like this.

~~~ruby
class Calendar
  def initialize(date=Date.today)
    @date = date
  end

  def to_a
    CalendarWeeks.new(@date).to_a.map do |week|
        [date, css_classes_for(date)]
    end
  end

private

  def css_classes_for(date)
   ["past", "other-month"].compact.join(" ")  
  
   "past other-month"
  end
  
end
~~~

Now, let's take this pseudo code and make it actually do the real thing. We can replace the string with a method. Let's start with just the past. If this is now a method that returns the string `past` if the date ... This means we need the date, both `past` and the date. Now, we `past` and the date. Let's compare that date to being less than `Date.today.` We had another one and it was `def other_month,` and this one would return `other-month` if the `date.month` does not equal the `Date.today.month.` That's going to need the date too. Now, we also need the `other_month` method, which takes that date. 

~~~ruby
 def css_classes_for(date)
   [past(date), other_month(date)].compact.join(" ")  
 end
 
 def past(date)
   "past" if < Date.today
 end
 
 def other_month(date)
    "other-month" if date.today != Date.today.month
 end
~~~

We could keep going down this route, but it's getting very convoluted. It's not going to read very well, `past(date), other_month(date).` It's got too much repeat of `date` in here. What this is telling me is that there's an object that wants to get out. The way I'm going to do this is I'm actually going to put an end on the previous class and create a new class. This is going to be called `DayStyles.` Then, let's move the private designator down here. Make this a public method. We're going to change that as a `to_s.` We're going to change the `DayStyles` object into a string, so we're going to use the `to_s.` This is going to have an initializer which is going to take a date. This time we're not going to allow an empty date, and we're going to save that date off into in instance variable. That's going to meant that we don't have to call `date` over and over here, which cleans up our code. We don't have to call it here. We actually have access to this instance variable date. Same thing here. It's now in the instance variable date.

~~~ruby
class Calendar
  def initialize(date=Date.today)
    @date = date
  end

  def to_a
    CalendarWeeks.new(@date).to_a.map do |week|
      week.map do |date|
        [date, css_classes_for(date).to_s]
      end
    end
  end
end

class DayStyles
  def initialize(date)
    @date = date
  end

  def to_s
    [past, other_month].compact.join(" ")
  end

private

  def past
    "past" if @date < Date.today
  end

  def other_month
    "other-month" if @date.month != Date.today.month
  end
end
~~~

Let's continue to build this out. We want to worry about past. We also need to know if it's today. We also need to know if it's in the future. The `future` and `today` methods look a lot like the `past` method. It's going to return the string `today` if the day is equal to this date. Then, we have the `future` method, which is just going to return the string `future if the date is greater than today.` Now, say that the date is today. Past is going to be nil. Today is going to be the string `today.` Future is going to be nil, and other_month is going to be nil. This is where the compact kicks in and flattens the array out so there are no nils. Then, we join the remaining values together with an empty space.

~~~ruby
class DayStyles
  def initialize(date)
    @date = date
  end

  def to_s
    [past, today, future, other_month].compact.join(" ")
  end

private

  def past
    "past" if @date < Date.today
  end

  def today
    "today" if @date == Date.today
  end

  def future
    "future" if @date > Date.today
  end

  def other_month
    "other-month" if @date.month != Date.today.month
  end
end
~~~

There's one last thing to do here. We changed this from being a method o the calendar object to being its own object called `DayStyles.` We're going to new it up, and we're going to call `to string` on it, which you can see here. The public method is `to string,` and that's where this happens. If we background this and rerun our code, we now get the date followed by some extra CSS information. If the calendar day is part of the last month, we have `other_month` and the word `past.` If it's part of the future month, we have `other_month` and `future,` and if you look in there for today, you'll see that today is indeed flagged with `today.`

~~~ruby
class Calendar
  def initialize(date=Date.today)
    @date = date
  end

  def to_a
    CalendarWeeks.new(@date).to_a.map do |week|
      week.map do |date|
        [date, DayStyles.new(date).to_s]
      end
    end
  end
end

class DayStyles
  def initialize(date)
    @date = date
  end

  def to_s
    [past, today, future, other_month].compact.join(" ")
  end

private

  def past
    "past" if @date < Date.today
  end

  def today
    "today" if @date == Date.today
  end

  def future
    "future" if @date > Date.today
  end

  def other_month
    "other-month" if @date.month != Date.today.month
  end
end
~~~

![Calendario](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/004/images/002.png)

Now, let's put this into a Rails app. In the next episode we're going to go over this Rails code in more detail, but I wanted to show you what this calendar is going to look like. Here's the show action that renders out the calendar. We're going to create a main element with he idea of `calendar` so that it matches up tot he style sheet. We're going to render out a partial called `headers` which just returns the days. Then, we're going to render out this partial week one for every week inside of our object, which happens to be six times. Inside of each of these we render out the day partial. Every time the day partial is rendered, it will be rendered with a dated date to the two digits in that day, so 26, 01, 02. The div will also have a class of day, and of those values that come from the object that we just created.

~~~ruby
# rubycast_calendar/app/views/calendars/show.html.erb

<main id="calendar">
  <%= render partial: 'headers' %>
  <%= render partial: 'week', collection: @calendar %>
</main>

# rubycast_calendar/app/views/calendars/_header.html.erb

<section class="th">
  <span>Sunday</span>
  <span>Monday</span>
  <span>Tuesday</span>
  <span>Wednesday</span>
  <span>Thursday</span>
  <span>Friday</span>
  <span>Saturday</span>
</section>

#rubycast_calendar/app/views/calendars/_week.html.erb 

<div class="week">
  <%= render partial: 'day', collection: week %>
</div>

#rubycast_calendar/app/views/calendars/_day.html.erb 
<div data-date="<%= day[0].strftime("%d") %>" class="day <%= day[1] %>">
</div>
~~~

The styling is handled by this style sheet. We can see here's the ID for the main calendar ID. The final thing to note before we see this is the div itself ... If we go back to the day partial, the div itself does not have any content inside of it. It has the div definition, but it has no content inside of that div. The day number is represented in the calendar by this line right here. We have a week class that's going to have a div inside of it, which is one of those days. We're going to set the content of the div into whatever's in this dated date. Without further adieu, ta da! Here's our calendar fully styled. If we are to right click the 26th, we can see that it does indeed have the styles of `day,` and `past,` and `other_month,` which is exactly what we wanted. If we check out today's date, which is the 11th, we can see that it is indeed flagged with the class of `day` and a class of `today.` 

![Calendario](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/004/images/003.png)

![Calendario](https://s3-us-west-2.amazonaws.com/rubycastio-assets-production/asciicasts/004/images/004.png)

That's all we have time for in this week's episode of RubyCasts, but next week we will go over the Rails code that displays this in more detail. We'll talk about the controller, the route, and how exactly does Rails know how to render what it renders. To top it off, we're going to throw on some CoffeeScript so that we can register events onto today's date and dates in the future, but not dates in the past. Next episode we're going to be doing the full stack from the Ruby, to the Rails, to the CoffeeScript. See you next time.
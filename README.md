Original plugin by tamoyal, modifications by Casey Watts and Chris
Oliver.

Changes:
*A default time can be set (especially useful for editing objects)
*Cross-midnight capability. Start and end times don’t need to be on the same day. An asterisk is placed after anything in “the next day” [12:30*].
*Start and end times can be any time; it doesn’t just round to the nearest hour.
*Start and end times can be excluded from the form if needed.

I feed these values to the form by using instance variables defined in a helper, which I call at the top of the view page.

Example use:
      <p>
        <%= f.label :start %><br />
        <%= f.time_select :start_time, {
                :simple_time_select => true,
                :time_separator => "",
                :minute_interval => @department.department_config.time_increment,

                :start_time => @range_start_time,
                :end_time => @range_end_time,

                :default => @time_slot.start,
                :include_start_time => true,
                :include_end_time => false,
                                          } %>
      </p>
      <p>
        <%= f.label :end %><br />
        <%= f.time_select :end_time, {
                :simple_time_select => true,
                :time_separator => "",
                :minute_interval => @department.department_config.time_increment,

                :start_time => @range_start_time,
                :end_time => @range_end_time,

                :default => @time_slot.end,
                :include_start_time => false,
                :include_end_time => true,
                                          } %>
      </p>

Controller code:

    def create
      # Parse the date and time fields into a new param and delete the individual params
      params[:event][:start_time] = Time.zone.parse("#{params[:event]["start_time(1i)"]}-#{params[:event]["start_time(2i)"]}-#{params[:event]["start_time(3i)"]} #{params[:event]["start_time(5i)"]}")
      (1..5).each { |i| params[:event].delete "start_time(#{i}i)" }

      # Parse the date and time fields into a new param and delete the individual params
      params[:event][:end_time] = Time.zone.parse("#{params[:event]["end_time(1i)"]}-#{params[:event]["end_time(2i)"]}-#{params[:event]["end_time(3i)"]} #{params[:event]["end_time(5i)"]}")
      (1..5).each { |i| params[:event].delete "end_time(#{i}i)" }

      @event = Event.new params[:event]

      # The rest of your controller code
    end


SIMPLE TIME SELECT PLUGIN
=========================

Ever wanted a time select component with only one select field? This simple plugin
gives you that component and allows you to set minute intervals. If you set your
minute interval to 15, you get options such as:
"6:00 PM", "6:15 PM", "6:30 PM", etc.

If no minute interval is specified, the control defaults to a 15 minute interval.
As you can see from the sample values above, this control also implements an AM/PM time
format.

Also note that this component is great if you ONLY want the time from a user. As is, the code
prevents the hidden date fields associated with the time_select helper from being created. This means
that a couple lines of code on the controller side are necessary to make this work. See USAGE for details.

USAGE:

To create the time select:

  <%= time_select "event", "time", { :default => Time.now.change(:hour => 21), :simple_time_select => true, :minute_interval => 20, :time_separator => "" } %>

Don't forget to include the time_separator option.  Otherwise you will get an extra colon outside of the select field.

Simple time select also takes a start_hour and end_hour option to be specified in military format (between 0-23).

<%= time_select "event", "time", { :default => Time.now.change(:hour => 21), :simple_time_select => true, :minute_interval => 20, :time_separator => "", :start_hour => 10, :end_hour => 14 } %>

The start hour behaves as you would expect but the end_hour may not.  If you specify the end_hour as 10, your time select will include 10:15, 10:30, 10:45.  So the end_hour sets the last hour that the time select will include. Email me if you don't like this.

When the time is submitted, you will have the value in params on the controller side as shown below:

  params[:event][:"time(i)"]

Simply add these lines of code to your handler to play nicely with ActiveRecord:

  params[:event][:time] = Time.parse(params[:event][:"time(i)"])
  params[:event].delete(:"time(i)")

Now the time will be correct, but the date will be the current date.  Here are a couple options for changing the date:

1) Change the date in your controller (replace anything in < > with your own variables):

  params[:event][:time] = params[:event][:time].change(:year => <year_var>, :day => <day_var>, :month => <month_var>)
  *I do not use this method, you may need to cast params[:event][:time] to a Time object before applying "change" to it.

2) Change the date in an ActiveRecord callback in your model.  This is my preferred method:

  def before_validation
    # Assuming date is your Date variable
    self.time = self.time.change(:year => date.year, :day => date.day, :month => date.month)
  end

Feel free to send any questions to tonyamoyal@gmail.com

Author: Tony Amoyal <tonyamoyal@gmail.com>

Homepage:
http://github.com/tamoyal/simple_time_select/tree/master

log_wrapper
===========

*Fork me:* https://github.com/renenw/log_wrapper

*Install me:* `gem install log_wrapper`

Basics
------

A trivial wrapper for the Ruby Logger that adds convenience methods for timing execution, for logging in json, and for grep'ing the current log.

		log = Log_Wrapper.new  	# it takes the same parameters as the ruby logger class initializer
		log.info "Startup complete"
		log.debug("About to run some code (payload is expected to be a hash, and benchmark times are added to the payload, as well as a stack trace and error information if things go wrong)", :payload => payload) do
			# some code that probably does something with payload
		end
		log.warn "Not good...", :my_var => 'xxx', :my_other_var => 'yyy'
		log.error "Another message", :guid => 'my unique id', :payload => payload, :other_optional_var => 'actually, they are all optional except the message' do
			# more interesting code
		end

It gets interesting...
----------------------

`array_of_search_lines = log.grep('search_string')`

And, even more interesting...
-----------------------------

###log_controller.rb

		class LogController < ApplicationController

		  def index
		    log = Log_Wrapper.new('/home/ubuntu/my_app/log.txt')
		    @log_detail = log.web_grep(params['guid'])
		  end

		end

###log.html.erb

		<div id="Log_Entries">

		  <div class="page-header">
		    <h1><%=@log_detail['guid'] %> <small>at <span class="time"><%= @log_detail['start_time'].to_i*1000 %></span> UTC</small></h1>
		  </div>

		  <table class="table table-bordered lead">
		  	<tbody>
		  		<tr>
		  			<td>Start Time&nbsp;</td><td><span class="time" data-time_format='HH:MM:ss'><%= @log_detail['start_time'].to_i*1000 %></span></td>
		  		</tr>
		  		<tr>
		  			<td>End Time</td><td><span class="time" data-time_format='HH:MM:ss'><%= @log_detail['end_time'].to_i*1000 %></span></td>
		  		</tr>
		  		<tr>
		  			<td>Duration</td><td><span><%= @log_detail['duration'].to_i*1000 %></span> seconds</td>
		  		</tr>
		  		<% if @log_detail['times'] %>
			  		<tr>
			  			<td>User</td><td><span><%= number_with_precision(@log_detail['times']['user']*1000, :precision => 2, :separator => '.', :delimiter => ' ') %></span> ms</td>
			  		</tr>
			  		<tr>
			  			<td>System</td><td><span><%= number_with_precision(@log_detail['times']['system']*1000, :precision => 2, :separator => '.', :delimiter => ' ') %></span> ms</td>
			  		</tr>
			  		<tr>
			  			<td>Clock</td><td><span><%= number_with_precision(@log_detail['times']['elapsed']*1000, :precision => 2, :separator => '.', :delimiter => ' ') %></span> ms</td>
			  		</tr>
		  		<% end %>
		  	</tbody>
		  </table>

		  <br/>

		  <table id="readings" class="table table-striped table-bordered table-condensed">
		    <thead>
		      <tr>
		        <th>&nbsp;</th>
		        <th>When?</th>
		        <th>What?</th>
		        <th>Message</th>
		        <th>Payload</th>
		        <th style="text-align: right">ms</th>
		      </tr>
		    </thead>
		    <tbody>
		      <%= render :partial => "line", :collection => @log_detail['log_lines'] %>
		    </tbody>
		  </table>

		</div> 


### _line.html.erb

		<%
			elapsed = 0
			elapsed ||= line['times']['elapsed']
		%>
		<tr class="log_line <%= (elapsed==@log_detail['slowest_duration'] ? 'error' : '')%> ">
		  <td><%= line['severity'] %></td>
		  <td>
		  	<span class="time" data-time_format='HH:MM:ss'><%= line['date'].to_i*1000 %></span>
		  </td>
		  <td class="source"><%= line['app'] %></td>
		  <td class="reading"><%= line['message'] %></td>
		  <td class="payload"><%= line['payload'] %></td>
		  <td class="duration" style="text-align: right"><%= number_with_precision(elapsed*1000, :precision => 2, :separator => '.', :delimiter => ' ') %></td>
		</tr>


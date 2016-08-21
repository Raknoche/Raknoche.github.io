---
layout: post
title: Making a MySQL GUI with Tkinter
comments: true
---

Introduction to MySQL databases and Python's Tkinter library.

<!--more-->

With the start of every school year comes a number of undergraduates interested in working in our physics lab.  In the past, new students had to learn how to interact with our MySQL databases before they could begin any data analysis.  This year, I decided to write a Python GUI that will make accessing and cleaning our data easier for new students.  The GUI will have a number of predefined query fields which the students can use, and will allow for visual inspection and cleaning of the data prior to saving the data to a local csv file. This approach will also reduce the number of students directly interacting with our MySQL database, and thereby reduce the risk of unintentional alterations to the data.


We'll cover the following topics:

1. [Introduction to MySQL](#MySQL)
2. [Making a Python GUI with Tkinter](#tkinter)
3. [Sending Queries from our GUI](#queries)
4. [Cleaning data with our GUI](#cleaning)
5. [Saving data with our GUI](#saving)


# <a name="MySQL"></a> Introduction to MySQL

MySQL is a free, open source [Relational Database Management System](https://en.wikipedia.org/wiki/Relational_database_management_system) (RDBMS) that uses [Structured Query Language](https://en.wikipedia.org/wiki/SQL) (SQL) to manage the content of various databases.

The SQL programming language is purpose-built for the manipulation of tabular data held in [Relational Database Management Systems](https://en.wikipedia.org/wiki/Relational_database_management_system) (RDBMS).  There are a number of RDBMS options available for use, including the free and open source [MySQL](https://www.mysql.com/) platform. 

You can install a MySQL database on your own computer with a few simple steps.  If you're a Mac or Linux user:

1. Download the latest DMG file from [MySQL](http://dev.mysql.com/downloads/mysql/)
2. Use the DMG file to install MySQL on your computer
3. Add the MySQL path to your system profile by typing `echo "export PATH=${PATH}:/usr/local/mysql/bin/" >> ~/.bash_profile` in the command line 
4. Configure a secure installation by typing `mysql_secure_installation` at the command line
5. Start MySQL by clicking apple $$\rarrow$$ system preferences $$\rarrow$$  MySQL $$\rarrow$$  Start MySQL Server

If your a Windows user, follow the installation directions on [MySQL's website](http://dev.mysql.com/downloads/windows/).  

You'll also want to install the Python MySQL library.  This allows you to interact with a MySQL database through Python.  Installing the library is as simple as typing `pip install pymysql` at the command line.

If you've never used SQL, I suggest using [Mode Analytics'](https://sqlschool.modeanalytics.com) or [SQLZoo's](sqlzoo.net/wiki/SQL_Tutorial) tutorials to familiarize yourself with the language.  You'll be able to follow this post without doing so, but you'll have trouble making a GUI for your own needs without knowledge of SQL.

# <a name="tkinter"></a> Making a Python GUI with Tkinter

Python's Tkinter library provides an easy to use interface to the Tk GUI toolkit.  To install Tkinter, type `pip install tkinter` at the command line.

Before we include any functionality in our GUI, lets build the basic layout.  First, we'll need to create a Tk GUI window in Python.  We'll pass this window to a customized class object (which I'm naming `App`) that will set up the layout our GUI in its `__init__` method.  Note that in this post any code that belongs to the `__init__` method will be indicated by two indents, just as it would be in our real Python code.  The `mainloop` method will open the window and display our GUI.

```python 
from tkinter import *

#Define a class to set up our GUI
class App(object):
    def __init__(self, window):
    	 #Set the window title
        window.wm_title("Database Interface")

#In our main function, create the GUI and pass it to our App class
def main():
	window= Tk()
	start= App(window)
	window.mainloop()

#Run the main function
if __name__ == "__main__":
    main()
```

Suppose our MySQL database contains data from a radioactivity counter.  The data table has the following columns

* `run_tag`: A string indicating the run which the data belongs to
* `clock_time`: A timestamp indicating when the radioactive decay occurred. Uses the format YYYY-MM-DD HH:MM:SS 
* `amplitude`: The energy of the observed decay

We'd like to select data from the database by providing our GUI a run tag, a start time, an end time, or any combination of the three.  To do so, we'll want to create some entry boxes in our GUI where the user can specify these parameters. I'll refer to this section of our GUI as the "query panel." 

In the first row of our query panel, we'll ask the user to specify a run tag.  We can create an empty class variable to store the run tag by using Tkinter's `StringVar()` function.  

```python
		self.runtag_text = StringVar()
```

To assign a string to the variable we use Tkinter's `Entry` object, passing the variable we just created as the second argument.  This will display an entry box on our GUI that our user can type run tag labels into.  

```python
		self.runtag_entry = Entry(window, textvariable=self.runtag_text)
```

To position the entry box on our GUI, we can use the `grid` method, specifying a row and a column.  For easy modification of the GUI code, I like to keep track of the current row using a `self.current_row` variable in the `App` class.     

```python
		self.current_row=0
		self.runtag_entry.grid(row=self.current_row,column=1)
```

We'll want display text on the GUI window that indicates the entry box we just created is to be used for run tags.  We can do so using Tkinter's `Label` object.

```python
		self.runtag_label = Label (window, text= "Run Tag: ")
		self.runtag_label.grid(row=self.current_row,column=0)
```
We can repeat this process for each of the three parameters, incrementing the `self.current_row` by one for each new entry box.  


At the bottom of our query panel, we'll want to add a button that will submit the user's query to our MySQL database.  I'll discuss how to implement the functionality of this button later in the post, but for now we can place it on the GUI using Tkinter's `Button` object.  We can indicate the purpose of the button by specifying the button's text in the second argument of the `Button` method.  Since the button will be on a row by itself, we can place it in between column zero and column one by specifying a `columnspan` of two when we use the `grid` method.
        
```python        
		self.query_button = Button (window, text="Send Query")
		self.query_button.grid(row=self.current_row,column=0,columnspan=2)
```

Eventually, we'll want the button to execute a method that we define in the `App` class.  Although we haven't written it yet, we'll call the method `sendQuery`.  We can make the button execute `sendQuery` when pressed by using the `configure` method.  I'll leave this particular line commented out for now, since the `sendQuery` method doesn't exist yet.

```python 
		#self.query_button.configure(command=self.sendQuery)
``` 
     
We should also add a `Label` object underneath the query button that displays the status of our query after it is sent.  By default we'll leave the query status as an empty string.  After completing our query panel code, our `App` class will look like this:

```python
class App(object):
    def __init__(self, window):

        #Set the window title 
        window.wm_title("Database Interface")
        
        #Set the current row to zero
        self.current_row=0
        
        '''Query Panel'''
        #Run tag field
        self.runtag_label = Label (window, text= "Run Tag: ")
        self.runtag_label.grid(row=self.current_row,column=0)
        self.runtag_text  = StringVar()
        self.runtag_entry = Entry(window, textvariable=self.runtag_text)
        self.runtag_entry.grid(row=self.current_row,column=1)
        self.current_row += 1

        #Start time field
        self.start_t_label = Label (window, text= "Start Time (YYYY-MM-DD HH:MM:SS): ")
        self.start_t_label.grid(row=self.current_row,column=0)          
        self.start_t_text  = StringVar()
        self.start_t_entry = Entry(window, textvariable=self.start_t_text)
        self.start_t_entry.grid(row=self.current_row,column=1)
        self.current_row  += 1
        
        #End time field
        self.end_t_label = Label (window, text= "End Time (YYYY-MM-DD HH:MM:SS): ")
        self.end_t_label.grid(row=self.current_row, column=0)          
        self.end_t_text  = StringVar()
        self.end_t_entry = Entry(window, textvariable=self.end_t_text)
        self.end_t_entry.grid(row=self.current_row,column=1)
        self.current_row += 1     
        
        #Send Query button
        self.query_button = Button (window, text="Send Query")
        #self.query_button.command=self.sendQuery
        self.query_button.grid(row=self.current_row,column=0,columnspan=2)
        self.current_row += 1

        #Query status
        self.querystatus  = Label(window, text='')
        self.querystatus.grid(row=self.current_row,column=0,columnspan=2)
        self.current_row += 1
```

If you run the main function at this point, it will produce the following (functionless) GUI.

![png](https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/GUI_QueryPanel.png){: .center-image }
      
# <a name="queries"></a> Sending queries from our GUI

In this section, we'll implement the `sendQuery` method that our query button will call when pressed.  For clarity, I'll indent code that belongs in the `App` class with one tab, and code that belongs in the `sendQuery` method of the `App` class with two tabs.  Code that belongs outside of the `App` class will not be indented, just as it would not be in our Python code.

 
First, we define the new method within the `App` class.  Within the method, we can obtain any text that our user put in the entry boxes by using each variable's `.get()` method.

```python
	def sendQuery(self):
		start_t = self.start_t_text.get()
		end_t = self.end_t_text.get()
		run_tag = self.runtag_text.get()
```

Now we need to build a MySQL query using the strings we just collected above.  It's possible our user didn't place any text in the entry boxes, in which case our MySQL query should default to collecting all the timestamps and energies of the decay events in our database.  Assuming our data is stored in a table named `summarydata`, the syntax for this query would be

```sql
SELECT clock_time, amplitude FROM summarydata WHERE amplitude IS NOT NULL
```

The `IS NOT NULL` condition is not strictly necessary for this default query.  It ensures we do not collect data from our table if the energy information of an event is missing.  Regardless of whether we actually have missing energy data, we should still include the `IS NOT NULL` condition for another reason &mdash; it allows us to add additional conditions to our query with `AND` statements.  For instance, if our user provides a start time and end time in the entry boxes, our query would become

```sql
SELECT clock_time, amplitude FROM summarydata WHERE amplitude IS NOT NULL AND clock_time BETWEEN start_time AND end_time
```

If we did not have the `IS NOT NULL` condition in our default query, we would have to determine which query condition was first, and only add `AND` to the subsequent conditions.

To build our entire MySQL query, we will start with empty strings which represent the time and run tag conditions of our query.


```python
	    #Default to not including the conditions in the query
	    run_tag_string = ''
	    time_string = ''
```

Next, we'll check each of the entry box variables to see if our user entered text in each box.  If text was entered in an entry box, we change the empty condition string to a MySQL condition, such as `time_string = 'AND clock_time > start_t'`.  Note that the time condition requires the use of SQL's `BETWEEN` syntax if the user specifies both the start and end time of the query.

```python   
	    #Toggle query conditions on if the GUI field isn't blank
	    
	    #Run tag condition
	    if run_tag != '':
	        run_tag_string = 'AND run_tag = "%s"' %(run_tag)
	        
	    #Time window condition (in order: both set, only start set, only end set)
	    if start_t != '' and end_t != '':
	        time_string = 'AND clock_time BETWEEN "%s" AND "%s"' %(start_t,end_t)
	    elif start_t != '':
	        time_string = 'AND clock_time > "%s"' %(start_t)
	    elif end_t != '':
	        time_string = 'AND clock_time < "%s"' %(end_t)    
```      

Once we have determined the additional conditions to add to our default query, we simply append them to our original string to produce the final MySQL query.
  
```python                  
	    #Setting the query
	    self.query = "SELECT clock_time, amplitude FROM summarydata WHERE amplitude IS NOT NULL %s %s" % (run_tag_string, time_string)
``` 

To send the query to our MySQL database, we need to import the PyMySQL library at the top of our code. We'll also be using the Pandas library to store the data that our query returns, so we should import that as well.

```python
import pymysql as mdb
import pandas as pd
```

We can use the `connect` method of the PyMySQL library to open a connection to our database.  we need to provide an IP address, a username, a password, and a database name for the connection to work.  
   
```python
	    con = mdb.connect(self.ip, self.user, self.pswd, self.db);
```

It would be a security risk to include this information in the code itself, so instead we'll need to have each item passed into our program as a command line argument, and in turn pass them into our `App` class.

```python
class App(object):
	def __init__(self, window,ip,user,pswd,db):
		self.ip=ip
		self.user=user
		self.pswd=pswd
		self.db=db
		
		#Rest of class code goes here...

def main(argv):
	'''
	Usage: python radonGUI.py databaseIP username password databaseTable
	'''

	ip = argv[1]
	user = argv[2]
	pswd = argv[3]
	db = argv[4]

	window= Tk()
	start= App(window,ip,user,pswd,db)
	window.mainloop()


if __name__ == "__main__":
    main(sys.argv)
```

Once our MySQL connection is established, we create a `cursor` object which we'll use to interact with the database.  We tell the cursor to execute our MySQL query using the `execute` method, and we can fetch the return of the query using the `cur.fetchall()` method.  The data is returned as a tuple of tuples, with the outer tuple representing the row of the data table and the inner tuple representing the column.  It's convenient to store the data in a Pandas data frame, which we can easily do using list comprehensions.  Note that the timestamp from the database is returned in the `YYYY-MM-DD HH:MM:SS` format, so we should use the `mktime` method from Python's time library to convert it to a more convenient unix timestamp.

```python
		with con:
		    cur = con.cursor()
			
		    #Send select statement
		    cur.execute(self.query)
			
		    #Fetch the data from our query
		    rows = cur.fetchall()
			
		    #Fill a pandas data frame with timestamps and amplitude info
		    self.df = pd.DataFrame()
		    self.df['timestamp']=[time.mktime(rows[i][0].timetuple()) for i in range(len(rows))]
		    self.df['amplitude'] = [rows[i][1] for i in range(len(rows))]
```

Finally, we should update the query status label to inform our user that we have finished collecting all the data.

```python
    	self.querystatus.configure(text = "Finished")
```

Now that we have a working `sendQuery` method, we need to uncomment the line which assigns the `sendQuery` method to our query button.  Our entire code now looks like this:

```python
from tkinter import *
import pymysql as mdb
import pandas as pd
import time
from time import sleep

class App(object):
	def __init__(self, window,ip,user,pswd,db):
		self.ip=ip
		self.user=user
		self.pswd=pswd
		self.db=db

		#Set the window title 
		window.wm_title("Database Interface")

		#Set the current row to zero
		self.current_row=0

		'''Query Panel'''
		#Run tag field
		self.runtag_label = Label (window, text= "Run Tag: ")
		self.runtag_label.grid(row=self.current_row,column=0)
		self.runtag_text  = StringVar()
		self.runtag_entry = Entry(window, textvariable=self.runtag_text)
		self.runtag_entry.grid(row=self.current_row,column=1)
		self.current_row += 1

		#Start time field
		self.start_t_label = Label (window, text= "Start Time (YYYY-MM-DD HH:MM:SS): ")
		self.start_t_label.grid(row=self.current_row,column=0)          
		self.start_t_text  = StringVar()
		self.start_t_entry = Entry(window, textvariable=self.start_t_text)
		self.start_t_entry.grid(row=self.current_row,column=1)
		self.current_row  += 1

		#End time field
		self.end_t_label = Label (window, text= "End Time (YYYY-MM-DD HH:MM:SS): ")
		self.end_t_label.grid(row=self.current_row, column=0)          
		self.end_t_text  = StringVar()
		self.end_t_entry = Entry(window, textvariable=self.end_t_text)
		self.end_t_entry.grid(row=self.current_row,column=1)
		self.current_row += 1     

		#Send Query button
		self.query_button = Button (window, text="Send Query")
		self.query_button.configure(command=self.sendQuery)
		self.query_button.grid(row=self.current_row,column=0,columnspan=2)
		self.current_row += 1

		#Query status
		self.querystatus  = Label(window, text='')
		self.querystatus.grid(row=self.current_row,column=0,columnspan=2)
		self.current_row += 1

	def sendQuery(self):
		start_t = self.start_t_text.get()
		end_t = self.end_t_text.get()
		run_tag = self.runtag_text.get()

		#Default to not including the conditions in the query
		run_tag_string = ''
		time_string = ''

		#Toggle query conditions on if the GUI field isn't blank

		#Run tag condition
		if run_tag != '':
		    run_tag_string = 'AND run_tag = "%s"' %(run_tag)
		    
		#Time window condition (in order: both set, only start set, only end set)
		if start_t != '' and end_t != '':
		    time_string = 'AND clock_time BETWEEN "%s" AND "%s"' %(start_t,end_t)
		elif start_t != '':
		    time_string = 'AND clock_time > "%s"' %(start_t)
		elif end_t != '':
		    time_string = 'AND clock_time < "%s"' %(end_t)           
		                
		#Setting the query
		self.query = "SELECT clock_time, amplitude FROM summarydata WHERE amplitude IS NOT NULL %s %s" % (run_tag_string, time_string)

		con = mdb.connect(self.ip, self.user, self.pswd, self.db);

		with con:
		    cur = con.cursor()

		    #Send select statement
		    cur.execute(self.query)

		    #Fetch the data from our query
		    rows = cur.fetchall()

		    #Fill a pandas data frame with timestamps and amplitude info
		    self.df = pd.DataFrame()
		    self.df['timestamp']=[time.mktime(rows[i][0].timetuple()) for i in range(len(rows))]
		    self.df['amplitude'] = [rows[i][1] for i in range(len(rows))]

		self.querystatus.configure(text = "Finished")

def main(argv):
	'''
	Usage: python radonGUI.py databaseIP username password databaseTable
	'''
	ip = argv[1]
	user = argv[2]
	pswd = argv[3]
	db = argv[4]

	window= Tk()
	start= App(window,ip,user,pswd,db)
	window.mainloop()

if __name__ == "__main__":
    main(sys.argv)	
```
# <a name="saving"></a> Saving data with our GUI

Now that our GUI has a functional query panel, we'd like to let our users save a the data their query returns to a local csv file.  We can set up the layout of our "saving panel" using the same methods that we did for our "query panel."  Within out `App` class' `__init__` method, we add the following code:

```python
		'''Saving Panel'''  
		#Line to separate panels
		canvas = Canvas(master=window, width=500, height=40)
		canvas.create_line(0, 20, 500, 20, fill="black")
		canvas.grid(row=self.current_row,column=0,columnspan=2)
		self.current_row += 1
		    
		#Label for Entry box
		self.save_label = Label (window, text= "Save Location: ")
		self.save_label.grid(row=self.current_row, column=0, columnspan=2)          
		self.current_row += 1 
		     
		#Entry box for save location  
		self.save_text = StringVar()
		self.save_entry = Entry(window, textvariable=self.save_text)
		self.save_entry.grid(row=self.current_row,column=0, columnspan=2)
		self.current_row += 1 

		self.save_button = Button (window, text="Save Data")
		#self.save_button.configure(command=self.save_to_csv)
		self.save_button.grid(row=self.current_row,column=0,columnspan=2)
		self.current_row += 1
```

The only unfamiliar piece of the code above is the first six lines.  These lines use Tkinter's `Canvas` and `create_line` methods to draw a solid black line that separates the cleaning panel from the query panel.  The canvas is positioned using the `grid` method that we are already familiar with.

If we run the main program at this point, the GUI will now look like this:

![png](https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/GUI_SavingPanel.png)


Now we need to write the `self.EvT_select`, `self.EvT_delete`, and `self.EvT_select_only` methods that correspond to the three buttons we just added.



# <a name="saving"></a> Saving data with our GUI



# Closing remarks

In the conclusions... link to the final code

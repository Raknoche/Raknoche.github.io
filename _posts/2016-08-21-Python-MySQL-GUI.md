---
layout: post
title: Making a MySQL GUI with Tkinter
comments: true
---

Introduction to MySQL databases and Python's Tkinter library.

<!--more-->

With the start of every school year comes a number of undergraduates interested in working in our physics lab.  In the past, new students had to learn how to interact with our MySQL databases before they could begin any data analysis.  This year, I decided to write a Python GUI that will make accessing and cleaning our data easier for new students.  The GUI will have a number of predefined query fields which the students can use to obtain data before saving that data to a local csv file. This approach will also reduce the number of students directly interacting with our MySQL database, and thereby reduce the risk of unintentional alterations to the data.


We'll cover the following topics:

1. [Introduction to MySQL](#MySQL)
2. [Making a Python GUI with Tkinter](#tkinter)
3. [Sending Queries from our GUI](#queries)
4. [Saving data with our GUI](#saving)


# <a name="MySQL"></a> Introduction to MySQL

MySQL is a free, open source [Relational Database Management System](https://en.wikipedia.org/wiki/Relational_database_management_system) (RDBMS) that uses [Structured Query Language](https://en.wikipedia.org/wiki/SQL) (SQL) to manage the content of various databases.  You can install a MySQL database on your own computer with a few simple steps.  If you're a Mac or Linux user:

1. Download the latest DMG file from [MySQL](http://dev.mysql.com/downloads/mysql/)
2. Use the DMG file to install MySQL on your computer
3. Add the MySQL path to your system profile by typing `echo "export PATH=${PATH}:/usr/local/mysql/bin/" >> ~/.bash_profile` at the command line 
4. Configure a secure installation by typing `mysql_secure_installation` at the command line
5. Start MySQL by clicking apple $$\rightarrow$$ system preferences $$\rightarrow$$  MySQL $$\rightarrow$$  Start MySQL Server

If your a Windows user, follow the installation directions on [MySQL's website](http://dev.mysql.com/downloads/windows/).  

You'll also want to install the Python MySQL library.  This allows you to interact with a MySQL database through Python.  Installing the library is as simple as typing `pip install pymysql` at the command line.

If you've never used SQL, I suggest following [Mode Analytics'](https://sqlschool.modeanalytics.com) or [SQLZoo's](sqlzoo.net/wiki/SQL_Tutorial) tutorials to familiarize yourself with the language.  You'll be able to follow this post without doing so, but you'll have trouble making a GUI for your own needs without knowledge of SQL.

# <a name="tkinter"></a> Making a Python GUI with Tkinter

Python's Tkinter library provides an easy to use interface to the Tk GUI toolkit.  We'll be using it to build our customized GUI. To install Tkinter, type `pip install tkinter` at the command line.

Before we include any functionality in our GUI, lets build the basic layout.  First, we'll need to create a Tk GUI window in Python.  We'll pass this window to a customized class object (which I'm naming `App`) that will set up the layout of our GUI in its `__init__` method.  Note that in this post any code that belongs to the `__init__` method will be indicated by two indents, just as it would appear in our real Python code.  The `mainloop` method will open the window and display our GUI.  

```python 
#We aren't using all of the libraries right now
#but we will use them later in the post

from tkinter import *
import pymysql as mdb
import pandas as pd
import time
from tkinter import messagebox
import datetime
import re
import os

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

* `run_tag`: A string indicating a collection which the data belongs to
* `clock_time`: A timestamp indicating when the radioactive decay occurred. Uses the format YYYY-MM-DD HH:MM:SS 
* `amplitude`: The energy of the observed decay

We'd like to select data from the database by providing our GUI a run tag, a start time, an end time, or any combination of the three.  To do so, we'll need to create some entry boxes in our GUI where the user can specify these parameters. I'll refer to this section of our GUI as the "query panel." 

In the first row of our query panel, we'll ask the user to specify a run tag.  We can create an empty class variable to store the run tag by using Tkinter's `StringVar()` function.  

```python
    	self.runtag_text = StringVar()
```

To assign a string to the variable we use Tkinter's `Entry` object, passing the variable we just created as the second argument.  This will display an entry box on our GUI that our user can type run tags into.  

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
We can repeat this process for each of the three query parameters, incrementing the `self.current_row` by one for each new entry box.  


At the bottom of our query panel, we'll want to add a button that will submit the user's query to our MySQL database.  I'll discuss how to implement the functionality of this button later in the post, but for now we can place it on the GUI using Tkinter's `Button` object.  We can indicate the purpose of the button by specifying the button's text in the second argument of the `Button` method.  Since the button will be on a row by itself, we can place it in between column zero and column one by specifying a `columnspan` of two when we use the `grid` method.
        
```python        
    	self.query_button = Button (window, text="Send Query")
    	self.query_button.grid(row=self.current_row,column=0,columnspan=2)
```

Eventually, we'll want the button to execute a method that we define in the `App` class.  Although we haven't written it yet, we'll call the method `sendQuery`.  We can make the button execute `sendQuery` when pressed by using the `configure` method.  I'll leave this particular line commented out for now, since the `sendQuery` method doesn't exist yet.

```python 
    	#self.query_button.configure(command=self.sendQuery)
``` 
     
We should also add a `Label` object underneath the query button that displays the status of our query after it is sent.  By default we'll set the query status as an empty string.  After completing our query panel code, our `App` class will look like this:

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

Before we use any text that was entered, we need to verify that it is a valid string to use in our query.  We can use Python's [regular expression](https://docs.python.org/2/library/re.html) library to confirm that the start time and end time are in the YYYY-MM-DD HH:MM:SS format (if they were entered at all).  If the time parameters are in the correct format, we'll also need to verify that the date specified is a valid date.  For instance, a start time of 2001-02-31 10:05:42 should be identified as a mistake since February does not have 31 days.  We can do so by trying to convert the string to a timestamp with `datetime.datetime`.  If the string is not a valid date, `datetime.datetime` will raise an exception which we can use to flag the date as being invalid.

```python
        #Check that date is valid
        time_format = re.compile('^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}$')
        start_t_okay=None
        if (start_t) and (time_format.match(start_t)):
            start_year=int(start_t[0:4])
            start_month=int(start_t[5:7])
            start_day=int(start_t[8:10])
            start_hour=int(start_t[11:13])
            start_min=int(start_t[14:16])
            start_sec=int(start_t[17:19])
            try:
                datetime.datetime(start_year,start_month,start_day,start_hour,start_min,start_sec)
                start_t_okay='yes'
            except:
                start_t_okay='no'


        end_t_okay=None
        if (end_t) and (time_format.match(end_t)):
            end_year=int(end_t[0:4])
            end_month=int(end_t[5:7])
            end_day=int(end_t[8:10])
            end_hour=int(end_t[11:13])
            end_min=int(end_t[14:16])
            end_sec=int(end_t[17:19])
            try:
                datetime.datetime(end_year,end_month,end_day,end_hour,end_min,end_sec)
                end_t_okay='yes'
            except:
                end_t_okay='no'
``` 

Now that we have determined if the start and end times are valid, we need to go through a list of possible errors before constructing our MySQL query.  Here's everything that could go wrong:

* The start time is not the correct format
* The end time is not the correct format
* The start time is not a valid date 
* The end time is not a valid date
* The start time doesn't come before the end time
* The run tag doesn't exist in our database

If any of these failure modes occur, we'll want to display a message to the user to inform them of the mistake.  We can do so using Tkinter's `messagebox.showinfo` method.

```python
        '''Error Handling'''
        #Check that the date is in the correct format
        if (start_t) and (not time_format.match(start_t)):
            messagebox.showinfo('Query Error', 'Error: Start time Must have the format "YYYY-MM-DD HH:MM:SS" or be empty')
        elif (end_t) and (not time_format.match(end_t)):
            messagebox.showinfo('Query Error', 'Error: End time Must have the format "YYYY-MM-DD HH:MM:SS" or be empty')
        #Check that the date is a valid date
        elif (start_t) and (start_t_okay == 'no'):        
            messagebox.showinfo('Query Error', 'Error: Start time is not valid')
        elif (end_t) and (end_t_okay == 'no'):
            messagebox.showinfo('Query Error', 'Error: End time is not valid')
        #Check that start_t < end_t
        elif (start_t) and (end_t) and (start_t > end_t):         
            messagebox.showinfo('Query Error', 'Start time must come before end time')    
        else:
            #Construct and send the query
```

You may have noticed we did not check if the specified run tag exists in our database.  Assuming our data is stored in a table named `summarydata`, we could get a list of all of the run tags by sending a `SELECT DISTINCT(run_tag) FROM summarydata` query to the database.  This query scans the entire database to collect a list of every run tag, so it can take a while to run.  Instead of explicitly checking this condition, it would be more efficient to send the query with a nonexistent run tag and inform the user that the query did not return any data.  We'll implement this later in the code.


Now that we've ensured our user has entered valid text, we need to build a MySQL query using that text.  It's possible our user didn't place any text in the entry boxes at all, in which case our MySQL query should default to collecting all the data in our database.  The syntax for this query would be

```sql
SELECT clock_time, amplitude FROM summarydata WHERE amplitude IS NOT NULL
```

The `IS NOT NULL` ensures we do not collect data from our table if the energy record of an event is missing.  Regardless of whether we actually have missing energy data, we should still include the `IS NOT NULL` condition for another reason &mdash; it allows us to add additional conditions to our query with `AND` statements.  For instance, if our user provides a start time and end time in the entry boxes, our query would become

```sql
SELECT clock_time, amplitude FROM summarydata WHERE amplitude IS NOT NULL AND clock_time BETWEEN start_time AND end_time
```

If we did not have the `IS NOT NULL` condition in our default query, we would have to determine which query condition was first, and only add `AND` to the subsequent conditions.

To include conditional statements in our MySQL query, we will start with empty strings which represent the time and run tag conditions of our query.


```python
        #error checking was done up here
		
        else:
            #Default to not including the conditions in the query
            run_tag_string = ''
            time_string = '''
```



If text was entered in an entry box, we change the corresponding condition string to a MySQL condition, such as `time_string = 'AND clock_time > start_t'`.  

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

To send the query to our MySQL database, we need to import the PyMySQL library at the top of our code.  We can use the `connect` method of the PyMySQL library to open a connection to our database.  We need to provide an IP address, a username, a password, and a database name for the connection to work.  
   
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

Once our MySQL connection is established, we create a `cursor` object which we'll use to interact with the database.  We tell the cursor to execute our MySQL query using the `execute` method, and we can fetch the return of the query using the `cur.fetchall()` method.  The data is returned as a tuple of tuples, with the outer tuple representing the row of the data table and the inner tuple representing the column.  It's convenient to store the data in a Pandas data frame, which we can easily do using list comprehensions.  Note that the timestamp from our database is returned in the `YYYY-MM-DD HH:MM:SS` format, so we should use the `mktime` method from Python's time library to convert it to a more convenient unix timestamp.

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

Once our query is complete, we should update the query status label to inform our user that we have finished collecting all the data.

```python
            self.querystatus.configure(text = "Finished")
```

Finally, we should check to see if our query actually returned data from out database.  If the data frame is empty, we should display a warning to our user that no data was collected from the query.  Note that this will also handle any issues with nonexistent run tags that our user may have entered earlier.

```python
            #Check if the query returned data
            if self.df.empty:
                 messagebox.showinfo('Query Warning', 'Warning: Query returned no data')
```


Now that we have a working `sendQuery` method, we need to uncomment the line which assigns the `sendQuery` method to our query button.  Our entire code now looks like this:

```python
from tkinter import *
import pymysql as mdb
import pandas as pd
import time
from tkinter import messagebox
import datetime
import re
import os

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

        #Check that date is valid
        time_format = re.compile('^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}$')
        start_t_okay=None
        if (start_t) and (time_format.match(start_t)):
            start_year=int(start_t[0:4])
            start_month=int(start_t[5:7])
            start_day=int(start_t[8:10])
            start_hour=int(start_t[11:13])
            start_min=int(start_t[14:16])
            start_sec=int(start_t[17:19])
            try:
                datetime.datetime(start_year,start_month,start_day,start_hour,start_min,start_sec)
                start_t_okay='yes'
            except:
                start_t_okay='no'


        end_t_okay=None
        if (end_t) and (time_format.match(end_t)):
            end_year=int(end_t[0:4])
            end_month=int(end_t[5:7])
            end_day=int(end_t[8:10])
            end_hour=int(end_t[11:13])
            end_min=int(end_t[14:16])
            end_sec=int(end_t[17:19])
            try:
                datetime.datetime(end_year,end_month,end_day,end_hour,end_min,end_sec)
                end_t_okay='yes'
            except:
                end_t_okay='no'        

        '''Error Handling'''
        #Check that the date is in the correct format
        if (start_t) and (not time_format.match(start_t)):
            messagebox.showinfo('Query Error', 'Error: Start time Must have the format "YYYY-MM-DD HH:MM:SS" or be empty')
        elif (end_t) and (not time_format.match(end_t)):
            messagebox.showinfo('Query Error', 'Error: End time Must have the format "YYYY-MM-DD HH:MM:SS" or be empty')
        #Check that the date is a valid date
        elif (start_t) and (start_t_okay == 'no'):        
            messagebox.showinfo('Query Error', 'Error: Start time is not valid')
        elif (end_t) and (end_t_okay == 'no'):
            messagebox.showinfo('Query Error', 'Error: End time is not valid')
        #Check that start_t < end_t
        elif (start_t) and (end_t) and (start_t > end_t):         
            messagebox.showinfo('Query Error', 'Start time must come before end time')    
        else:
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

            #Check if the query returned data
            if self.df.empty:
                 messagebox.showinfo('Query Warning', 'Warning: Query returned no data')
           

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

Now that our GUI has a functional query panel, we'd like to let our users save the data to a local csv file.  We can set up the layout of our "saving panel" using the same methods that we did for our query panel.  Within the `App` class' `__init__` method, we add the following code:

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

The only unfamiliar piece of the code above is the first six lines.  These lines use Tkinter's `Canvas` and `create_line` methods to draw a solid black line that separates the saving panel from the query panel.  The canvas is positioned using the `grid` method that we are already familiar with.

If we run the main program at this point, the GUI will look like this:

![png](https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/GUI_SavingPanel.png)


Now we need to write the `self.save_to_csv` method that corresponds to the button we just added.  We define this new method within the `App` class, and use the familiar `get()` method from Tkinter to obtain the save location from the entry box we just created.  We'll need to check that the directory specified in the save path exists, and that the path ends with an appropriate `filename.csv` filename.  We should also verify that the data frame is not empty, since we can not save data that doesn't exist.  Once we have an appropriate save location, Pandas provides a convenient `to_csv` method to save the data frame to a csv file.

```python
    def save_to_csv(self):
        #Verify that the data frame is not empty
        if self.df.empty:
             messagebox.showinfo('Save Error', 'Error: Can not save empty data frame')

        #If we have data
        else:
            #Get the save location and extract the directory path
            self.save_loc = self.save_text.get() 
            self.save_dir = os.sep.join(self.save_loc.split(os.sep)[:-1])

            #Verify that the directory exists
            if os.path.isdir(self.save_dir):

                #Verify that the filename is more than just '.csv'
                if len(self.save_loc.split(os.sep)[-1]) < 5:
                    messagebox.showinfo('Save Error', 'Error: Path should end in filename.csv')

                #Verify that the filename ends in '.csv'
                elif self.save_loc.split(os.sep)[-1][-4:] != '.csv':
                    messagebox.showinfo('Save Error', 'Error: File should end in filename.csv')

                #Save the data if it passes error checking
                else:
                    self.df.to_csv(self.save_loc)
                    
            #Error if the directory doesn't exist        
            else:
                messagebox.showinfo('Save Error', 'Error: File directory does not exist')
```

After uncommenting the line which assigns functionality to the save button, our final GUI code looks like this:

```python
from tkinter import *
import pymysql as mdb
import pandas as pd
import time
from tkinter import messagebox
import datetime
import re
import os

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
        self.save_button.configure(command=self.save_to_csv)
        self.save_button.grid(row=self.current_row,column=0,columnspan=2)
        self.current_row += 1


    def sendQuery(self):
        start_t = self.start_t_text.get()
        end_t = self.end_t_text.get()
        run_tag = self.runtag_text.get()

        #Check that date is valid
        time_format = re.compile('^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}$')
        start_t_okay=None
        if (start_t) and (time_format.match(start_t)):
            start_year=int(start_t[0:4])
            start_month=int(start_t[5:7])
            start_day=int(start_t[8:10])
            start_hour=int(start_t[11:13])
            start_min=int(start_t[14:16])
            start_sec=int(start_t[17:19])
            try:
                datetime.datetime(start_year,start_month,start_day,start_hour,start_min,start_sec)
                start_t_okay='yes'
            except:
                start_t_okay='no'


        end_t_okay=None
        if (end_t) and (time_format.match(end_t)):
            end_year=int(end_t[0:4])
            end_month=int(end_t[5:7])
            end_day=int(end_t[8:10])
            end_hour=int(end_t[11:13])
            end_min=int(end_t[14:16])
            end_sec=int(end_t[17:19])
            try:
                datetime.datetime(end_year,end_month,end_day,end_hour,end_min,end_sec)
                end_t_okay='yes'
            except:
                end_t_okay='no'        

        '''Error Handling'''
        #Check that the date is in the correct format
        if (start_t) and (not time_format.match(start_t)):
            messagebox.showinfo('Query Error', 'Error: Start time Must have the format "YYYY-MM-DD HH:MM:SS" or be empty')
        elif (end_t) and (not time_format.match(end_t)):
            messagebox.showinfo('Query Error', 'Error: End time Must have the format "YYYY-MM-DD HH:MM:SS" or be empty')
        #Check that the date is a valid date
        elif (start_t) and (start_t_okay == 'no'):        
            messagebox.showinfo('Query Error', 'Error: Start time is not valid')
        elif (end_t) and (end_t_okay == 'no'):
            messagebox.showinfo('Query Error', 'Error: End time is not valid')
        #Check that start_t < end_t
        elif (start_t) and (end_t) and (start_t > end_t):         
            messagebox.showinfo('Query Error', 'Start time must come before end time')    
        else:
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

            #Check if the query returned data
            if self.df.empty:
                 messagebox.showinfo('Query Warning', 'Warning: Query returned no data')
           


    def save_to_csv(self):
        #Verify that the data frame is not empty
        if self.df.empty:
             messagebox.showinfo('Save Error', 'Error: Can not save empty data frame')

        #If we have data
        else:
            #Get the save location and extract the directory path
            self.save_loc = self.save_text.get() 
            self.save_dir = os.sep.join(self.save_loc.split(os.sep)[:-1])

            #Verify that the directory exists
            if os.path.isdir(self.save_dir):

                #Verify that the filename is more than just '.csv'
                if len(self.save_loc.split(os.sep)[-1]) < 5:
                    messagebox.showinfo('Save Error', 'Error: Path should end in filename.csv')

                #Verify that the filename ends in '.csv'
                elif self.save_loc.split(os.sep)[-1][-4:] != '.csv':
                    messagebox.showinfo('Save Error', 'Error: File should end in filename.csv')

                #Save the data if it passes error checking
                else:
                    self.df.to_csv(self.save_loc)
                    
            #Error if the directory doesn't exist        
            else:
                messagebox.showinfo('Save Error', 'Error: File directory does not exist')



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

# Closing remarks

The GUI we built in this post is a simplified version of the code I wrote for our new undergraduate students.  The complete GUI includes advanced functionality, such as the ability to change the units of our timestamps, plot the data in multiple ways at the push of a button, and select and delete data from those plots before saving the csv file.  These functions won't be applicable to everybody who reads this post, as they are tuned to the specific type of data my GUI collects.  If you'd like to learn how to implement similar functionality, feel free to look at the [complete code](https://github.com/Raknoche/MyCode/blob/master/CodeForDealingData/radonGUI.py) on my Github page and ask any questions you may have here.  
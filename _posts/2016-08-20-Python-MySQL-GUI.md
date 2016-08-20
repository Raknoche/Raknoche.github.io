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
3. Creating a MySQL GUI
4. Cleaning data with the GUI


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

Before we include any functionality in our GUI, lets build the basic layout.  First, we'll need to create a Tk GUI window in Python.  We'll pass this window to a customized class object (which I'm naming `App`) that will set up the layout our GUI in its `__init__` method. 

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

Suppose our (fictional) MySQL database contains data from a radioactivity counter.  The database has the following columns

* `run_tag`: A string indicating the run which the data belongs to
* `clock_time`: A timestamp indicating when the radioactive decay occurred. Uses the format YYYY-MM-DD HH:MM:SS 
* `amplitude`: The energy of the observed decay

We'd like to select data from the database by providing our GUI a run tag, a start time, an end time, or any combination of the three.  To do so, we'll want to create some entry boxes in our GUI where the user can specify these parameters. I'll refer to this section of our GUI as the "Query Panel." 

In the first row of our Query Panel, we'll ask the user to specify a run tag.  We can create an empty class variable to store the run tag by using Tkinter's `StringVar()` function.  

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


At the bottom of our Query Panel, we'll want to add a button that will submit the user's query to our MySQL database.  I'll discuss how to implement the functionality of this button later in the post, but for now we can place it on the GUI using Tkinter's `Button` object.  We can indicate the purpose of the button by specifying the button's text in the second argument of the `Button` method.  Since the button will be on a row by itself, we can place it in between column zero and column one by specifying a `columnspan` of two when we use the `grid` method.
        
```python        
self.query_button = Button (window, text="Send Query")
self.query_button.grid(row=self.current_row,column=0,columnspan=2)
```

Eventually, we'll want the button to execute a method that we define in the `App` class.  Although we haven't written it yet, we'll call the method `sendQuery`.  We can make the button execute `sendQuery` when pressed by using the `command` method.  I'll leave this particular line commented out for now, since the `sendQuery` method doesn't exist yet.

```python 
#self.query_button.command=self.sendQuery
``` 
     
After completing our Query panel code, our `App` class will look like this:

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
      
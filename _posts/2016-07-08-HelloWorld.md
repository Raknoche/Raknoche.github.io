---
layout: post
title: Hello, World!
comments: true
---

An introduction to the author, his transition from physics to data science, and the goals of this blog.

<!--more-->

# Who am I?

Hello, World! My name is Richard Knoche.  I'm a graduate student working on my Ph.D. in Physics at the University of Maryland.  My research focuses on advancing signal corrections and calibration techniques for the the world's most sensitive dark matter detector, [LUX](http://luxdarkmatter.org/).  While I enjoy the deep understanding of nature and the challenging experiences that a career in physics brings, I've also found a passion for using data analysis to contribute to the decision making process of our large collaboration.  To align my career with this passion, I hope to transition into the field of data science after I finish my Ph.D. at the end of 2016.

# What is a data scientist?

The term "data science" casts a wide net over a deep pool of specialties.  At the shallowest level, data scientists use math, computer science, and intuition to extract meaningful insights into large amounts of messy data.  If you dive a little deeper you'll find that this job description can mean different things to different companies.  At times, a data scientist is a glorified data analyst &mdash; interacting with MySQL databases and producing basic data visualizations in an effort to make the data more accessible and meaningful to their company.  Other times, a data scientist can take on more of a data engineering role.  In this case, they will be responsible for developing the company's data infrastructure and building pipelines of filtered information.  A data scientist can also be the person interacting with that filtered information &mdash; performing complex statistical and machine learning analyses to create actionable insights and data-driven products.

# How is data science related to physics?

The responsibilities of a data scientist bear an uncanny resemblance to my every day work as a physics graduate student.  My largest contribution to [LUX](http://luxdarkmatter.org/) was the construction of a signal corrections module.  This required me to clean up terabytes of calibration data on a remote server and apply various modeling techniques to automatically extract over 300 relevant quantities from each data set.  After extracting these quantities, I collected them in a MySQL database and produced streamlined scripts that helped collaboration members access the information on demand.  I also used the MySQL database to produce data visualizations that monitored the stability of our detector and informed day-to-day operations.  Many of my other responsibilities involved applying advanced statistical techniques such as linear regression, reduced $$\chi^2$$ fitting, Markov-Chain Monte Carlo methods, and maximum likelihood fitting to measure fundamental detector parameters such as our signal gains and energy scale calibration.  Throughout this process I used a combination of the Python, Matlab, and C++ programming languages, which are all popular among data scientists.

In addition to the aforementioned techniques, my time as a physics graduate student has taught me how to present complicated results to a wide variety of audiences.  I've written  technical, peer-reviewed papers for experts in the field, presented high-level overviews of [LUX](http://luxdarkmatter.org/) to physicists outside of dark matter research, and produced simple and digestible talks for the general public.  These skills are invaluable to data scientists who must translate complicated data results into informative presentations for their business counterparts.

# Why am I leaving physics?

If data science is so similar to physics, you might wonder why I want to change careers at all.  The truth is, I'd be happy staying in physics.  The challenging analysis and interpretation of data is exactly what I'm looking for in a career.  However, as an academic career progresses more and more of your time is taken away from data analysis tasks and instead spent writing grants and papers, and teaching classes.  Transitioning to a career in data science would allow me to maintain a focus on data analysis.  Below, I've included a graphic from [Alex Smolyanskaya](http://multithreaded.stitchfix.com/blog/2015/09/22/quantifying-my-transition-from-academia-to-data-science/) that illustrates this point.  I must admit, I'll miss the large amount of time that is dedicated to experiments during a post-doc position!

![Source: Alex Smolyanskaya](https://raw.githubusercontent.com/Raknoche/Raknoche.github.io/master/_posts/Images/Multithreaded-hours-timeline.png "Source: Alex Smolyanskaya")
Source: [Alex Smolyanskaya](http://multithreaded.stitchfix.com/blog/2015/09/22/quantifying-my-transition-from-academia-to-data-science/) 



# Data science fellowships

I've spent a lot of time thinking about the best way to break into the data science industry. I already have a strong quantitative background and am confident in my ability to contribute to a large project through data analysis.  It's likely that I'd be able to land a data science job at a small company and build a portfolio while there.  However, my academic career up until now has been just that &mdash; academic.  Without industry experience, or connections that help me bridge the gap, I suspect I'd have to make some compromises in my job responsibilities or my place of employment before I prove my skills in a new setting.

Luckily, a number of fellowships exist that help transition new post-docs from a career in academy to a career in data science.  The three programs I have looked at closely are [Insight Data Science](http://www.insightdatascience.com/), [The Data Incubator](https://www.thedataincubator.com/), and [DS12](http://education.datascience.com/).  Over a 7-12 week period, these programs put post-docs through a rigorous series of job training and portfolio building.  While some programs offer bootcamp modules that ensure their fellows are familiar with important data science techniques, others adopt the "learn by doing" mentality that post-docs in academy are all too familiar with.  (In reality, anybody who gets into these programs should be familiar with the data science techniques already.) Most importantly, the fellowships provide a foot in the door with industry leaders that academic post-docs would otherwise lack.  

# How have I been preparing for data science?

All of the programs I mentioned above are free.  They make their money through head-hunter fees and are mainly interested in post-docs who already have what it takes to succeed in data science.  As such, they are extremely competitive. 

I've spent the last four months researching, learning, and applying new skills to bolster my chances of getting into a data science fellowship.  I started by creating a list of important skills and resources to learn from.  In the end, the list came out similar to Insight's ["Preparing for Insight"](http://insightdatascience.com/blog/preparing_for_insight.html) blog post:


* Python:
     * [Code Academy](https://www.codecademy.com/learn/python) 
     * [Google Python Class](https://developers.google.com/edu/python/?csw=1)
     * [Anaconda and Jupyter notebook](jupyter.org/)
	 * [Pandas](https://github.com/estimate/pandas-exercises), [SciPy](http://www.engr.ucsb.edu/~shell/che210d/numpy.pdf), [Numpy](http://www.engr.ucsb.edu/~shell/che210d/numpy.pdf), [matplotlib](https://www.youtube.com/watch?v=q7Bo_J8x_dw), [Bokeh](http://nbviewer.jupyter.org/github/bokeh/bokeh-notebooks/blob/master/tutorial/00%20-%20intro.ipynb), [scikit-learn](http://scikit-learn.org/stable/), [NLTK](http://www.nltk.org/book/)


* MySQL:
    * [Mode Analytics](https://sqlschool.modeanalytics.com)
    * [SQLZoo](sqlzoo.net/wiki/SQL_Tutorial)
    * [MySQL Python Tutorial](zetcode.com/db/mysqlpython/)


* Machine Learning:
    * [Andrew Ng's Course](https://www.coursera.org/learn/machine-learning)
    * [Python Machine Learning](https://www.amazon.com/Python-Machine-Learning-Sebastian-Raschka-ebook/dp/B00YSILNL0#navbar)
	* [Applied Data Mining and Statistical Learning](https://onlinecourses.science.psu.edu/stat857/)

* Computer Science:
   * [Algorithms and Data Structures](interactivepython.org/runestone/static/pythonds/index.html)
   * [MIT Algorithms Course](https://www.youtube.com/playlist?list=PLUl4u3cNGP61Oq3tWYp6V_F-5jb5L2iHb)
   * [Leetcode problems](https://leetcode.com)

*  Statistics: 
   * [Applied Statistics](https://onlinecourses.science.psu.edu/stat500/)
   * [Regression Methods](https://onlinecourses.science.psu.edu/stat501/)
   * [Combinatorics](https://www.dartmouth.edu/~chance/teaching_aids/books_articles/probability_book/Chapter3.pdf)


* Interviewing:
   * [Nailing the Tech Interview](http://insightdatascience.com/blog/nailing-the-tech-interview-jessica-kirkpatrick.html)
   * [Programming Interview Examples](https://javarevisited.blogspot.com/2011/06/top-programming-interview-questions.html)
   * [More example problems](http://www.techinterview.org/)

* Data Science News and Information:
   * [Hacker News](https://news.ycombinator.com/)
   * [DataTau](www.datatau.com/)
   * [Quora](https://www.quora.com/topic/Data-Science/top_stories)
   * [Blogs](http://www.kdnuggets.com/2015/10/best-blogs-analytics-big-data-science-machine-learning.html)

This is a long list to learn in a few months.  Luckily, I was able to move through much of the content quickly since I was familiar with it from my graduate school work.  At this time I've finished the Python, MySQL, Machine Learning, and Statistics sections.  I've put some of the computer science algorithms and interviewing topics aside for now, since those will become more critical when applying to data science jobs directly after a fellowship.

If you use this list as a starting point for your own studies, be sure not to neglect the section on data science news and information.  Staying up to date on what others are doing in the field is critical when guiding your own career development.

# What do I want out of this blog?

As a closing note, I'd like to discuss my goals for this blog.  I want to help aspiring data scientists like myself learn the skills I outlined in the previous section.  To achieve this, I'll be approaching each blog post in a somewhat unorthodox manner.  Rather than choosing the best data analysis technique to answer a given question, I'll be searching for data which I can use to illustrate the techniques themselves.  In this way, the data analysis process will be the focus of each blog post, and the conclusions from those analyses will be of secondary importance.  It's my hope that this process will fortify my own understanding of each topic while providing a detailed introduction for my readers.


---
layout: post
title: "Observable Dictionary FTW"
date: 2015-02-27
---
NSS Cohort 7 has recently been working on making Windows desktop applications in WPF (Windows Presentation Foundation). Mine was a simple
note-taking app which organizes notes by category and date. I used a couple of ComboBoxes (what the rest of the world calls dropdowns) 
to select categories and search types, and since I wanted a particular selected item in the ComboBox to point to a particular value, I 
wanted to use a dictionary of key-value pairs to populate it.

Data binding a ComboBox to a dictionary was straightforward enough, but I soon found out that it was not a dynamic binding. If I added another category to the database, the ComboBox only reflected that additional category after I restarted the program. 

I soon found out that dictionaries in C# were by nature not observable collections, meaning that a UI element using the dictionary 
as a datasource will not be &ldquo;notified&rdquo; when the dictionary changes. An observable collection sends out change notifications 
for a particular index, but dictionaries are not indexed but rather an unorganized collection of key-value pairs. 

Enter the observable dictionary! I eventually found it on [Dr. Wpf&rsquo;s blog](http://drwpf.com/blog/2007/09/16/can-i-bind-my-itemscontrol-to-a-dictionary/). He constructed an Observable Dictionary class which basically *does* index a dictionary, which sends out change notifications. The drawback to this is that an observable dictionary is slower than a vanilla dictionary (apparently dictionaries are very fast) and 
requires more overhead, but that&rsquo;s perfectly fine for my application. Once I added the ObservableDictionary.cs file, all I had to do in 
the MainWindow class was make a static observable dictionary like so:

    public static ObservableDictionary<string, int> titleDict = new ObservableDictionary<string, int>(repo.GetAllTitles());

Now, adding a new note instantly updates the observable dictionary and that addition is reflected in the MainWindow. Clicking on the 
title (the key of type string) accesses the dictionary and gives the unique database id so we can retrieve that particular note. 

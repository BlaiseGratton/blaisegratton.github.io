---
layout: post
title: "Javascript Appending"
date: 2014-10-16 14:45
---

I have only been working with Javascript for a short time, so my current MO is to verbally walk through a task of Javascript which we have written to be sure I fully understand what is happening. Before lunch today we made a simple button which would append (i.e. add an element) text inside an element. The link to the source documents can be found here: [DOM Appending](https://github.com/sscotth/NSS-DomAppending) but I will also post the Javascript on its own here:


        1. document.addEventListener('DOMContentLoaded', function(){
        2.     var $button = document.querySelector('button');
        3. 
        4.     $button.addEventListener('click', function(){
        5. 	      var $target = document.querySelector('.target');
        6. 	      var docFragment = createPTag();
        7. 	      $target.appendChild(docFragment);
        8.     });
        9. });
        10. 
        11. function createPTag(){
        12.     var docFragment = document.createDocumentFragment();
        13. 
        14.     var $div = document.createElement('div');
        15.     $div.setAttribute('class', 'myClass');
        16.     var $p = document.createElement('p');
        17.     $div.appendChild($p);
        18.     $p.textContent = 'Hi';
        19.     docFragment.appendChild($div);
        20. 
        21.     return docFragment;
        22. }

Taking a step back, we have two overarching functions interacting: the click-event-listener on line 4 and the function createPTag on line 11. createPTag is subordinate to click-event-listener because it is not being called except from within click-event-listener at line 6. So what initially happens when running the program (assuming the DOM content is loaded correctly) is at line 2, the variable $button is assigned to the <button>Append</button> in the html document. Line 4 then makes Javascript wait for the button to be clicked. 

Once $button is clicked it is no longer important in the execution of the function. On line 5, click-event-function then creates a new variable $target which selects the div element in the html document with the class of &ldquo;target&rdquo;. This variable is set aside until line 7 where it only directs any further implementations of $target to the main div element on the html page.

Line 6 is where the two &ldquo;main&rdquo; functions interact. We are still in the process of running through the code from the click-event, but line 6 is calling for information returned from the createPTag function and equating this information with the variable docFragment on line 6. We therefore need to run through what the function createPTag returns.

It is important to keep in mind the issue of variable scope here. There are two variables called docFragment (ll. 6 &amp; 12), but none are declared on the global scope and both are independently maintained within their respective functions. Therefore, a new variable docFragment is being called on line 12, completely separate from the first docFragment variable. 

The createDocumentFragment() method on line 12 instructs Javascript to assemble elements together and insert them into a document. Before we call the actual function (rather than just declaring it in line 12), lines 14-18 are iterating what pieces are being assembled together. Thus, line 14 is creating a variable $div which is creating an actual html div element; line 15 is assigning the class of the div to be &ldquo;myClass&rdquo;. Line 16 is creating the variable $p to be a <p> element, with the content of &ldquo;Hi&rdquo; (line 18); line 17 inserts it as a child element to the div element created by the variable $div. So the end result of the compiled html code is:

	<div class="myClass"><p>Hi</p></div>

This is what is now contained by the variable $div. Running appendChild on the variable docFragment in line 19 (which is really document.createDocumentFragment from line 12) consequently adds the variable $div as the compiled document fragment, which line 21 returns as the output of the function. 

So the document fragment <div class="myClass"><p>Hi</p></div> is what is returned back to line 6, which the variable docFragment on line 6 is then equated to. Going down to line 7 tells Javascript to add the document fragment as a child to the $target variable, which is the main div element in the html document. This results in the div.target now containing the div.class which has the paragraph element displaying &ldquo;Hi&rdquo;.

This probably seems like way too much explaining for a relatively simple task, but even talking through it like this has greatly helped my understanding of how Javascript begins to act on the DOM.


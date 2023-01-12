---
layout: post
title:  "Software QA Automation/Testing (Java)"
date:   2021-07-16 14:34:25
categories: mediator feature
tags: 
image: /assets/article_images/2014-11-30-mediator_features/night-track.JPG
image2: /assets/article_images/2014-11-30-mediator_features/night-track-mobile.JPG
---
## QA Automation Tests using Java and JUnit suites in Eclipse.

Task was to build test cases centered around an application that creates objects such as appointments, tasks, and contacts. Questions that will be answered throughout the software lifecycle were: How can i ensure that my code, program, and software is functional and secure? How do I interpret user needs and incorporate them into a program? How would I approach software design?


#### Contact Class Requirements:


	• The contact object shall have a required unique contact ID string that cannot be longer than 10 characters. 

	• Contact ID cannot be null and cannot be updated.

	• Required firstName string that doesn’t exceed 10 characters. Cannot be null.

	• Required lastName string that doesn’t exceed 10 characters. Cannot be null.

	• Required phone string filed that must be (=) to exactly 10 digits. Cannot be null.

	• Required address field cannot be longer than 30 characters. cannot be null.


#### Contact Service Requirements:


	• Contact service shall add contacts with a unique ID.

	• Contact service shall be able to delete contacts per contact ID.

	• Contact service shall be able to update contact fields per contact ID: the following fields are: firstName, lastName, (Phone) Number, Address.


#### Task Class Requirements:


	• Task object shall have a required unique ID string that cannot be longer than 10 char*. cannot be null.

	• Task object shall have a required name String filed that cannot be longer than 20 char*. Cannot be null.

	• Task object shall have a required description String filed that cannot be longer than 50mchar*. cannot be null.


#### Task Service Requirements:


	• Task service shall be able to add tasks with a unique ID.

	• Task service shall be able to delete tasks per ask ID.

	• Task service shall be able to update task fields per task ID. 

	• Following fields are updateable: Name, Description.


### Methodology


My first action was to set up simple pseudocodes for the tests using the same design patterns in the live code. This was to ensure a reusable code process throughout multiple instances, provide a definitive solution(s) for the systems architecture, and provide transparency across all development teams. While this didn’t completely solve all problems, it allowed the testing team to provide clarity to the system and the possibility of building an optimized version. I noticed the source code was using creational design patterns such as Factory patterns and Abstract Factory patterns. This was used during the analysis and requirement phase of the SDLC. In addition, I aimed to keep the test cases and actual code symmetric, so the test cases use a lot of similar patterns as the production code. I should also add that I avoided grouping because of the increased complexity of the requirements. This was to (1. Added additional steps would prevent problems going forward (2. Individual test cases would result in higher percentage of coverage.


### Actions


The contact object class achieved 100% test coverage, the contact service had a majority of tests 100%, Task object class had 66.3% coverage. The task object class was much more complex in design and therefore, such as the generateUniqueID() class. 
For example, this is where Java design patterns shined as I was able to manufacture new tests for helper methods using the Abstract Factory pattern, along with getters and setters to handle the bulk of exceptions. 


    // setter example
    public void setFirstName (String firstName) {
	      checkIfGreaterThanTenOrNull(firstName);
	      this.firstName = firstName;
     }

    //setter example #2
    public void setPhoneNumber (String phoneNumber) {
	      checkifPhoneTenAndNotNull(phoneNumber);
	      this.phoneNumber = phoneNumber;
    }
  
  
### Results


    ContactTest
	    testAddress()
	    lastNameUpdateable()
	    idGettable()
	    addressUpdateable()
	    testLastNull()
	    testPhoneNull()
	    addressExceedsThirty()
	    firstNameUpdateable()
	    firstNameExceedsTen()
	    phoneNumberNotTen()
	    idExceedsTen()
	    phoneNumberlessThanTen()
	    testFirstNameNull()
	    lastNameExceedsTen()
	    testIdNull()
	    phoneNumberUpdateable()
	
    TaskService:
	    testAddTask()
	    testUpdateTask()
	    testUniqueId()
	    testDeleteTask()
	    testReadTaskNotFound()

    TaskTest: 
	    testNameIsNull()
	    testNameExceeds()
	    testIdExceeds()
	    testTask()
	    testDescriptionExceeds()
	    testDescriptionNull()
	    testSetTaskName()
	    testSetTaskDescription()
      
 
### Summary


This was an important project for learning the process of QA testing and the integration of QA with the SDLC among teams. For example, a big concern for me was the process of eliminating bias because, as developers, we want our code to be right and be the standard. Using Java design patterns did aid in the process as it tends to eliminate personal decision-making and instead focuses on transparency code and implementation of the current model. Software testing is a crucial part of the SDLC because most firms adopt an agile methodology, and tend to increase the software testing in more iterative increments whereas the classical model, the waterfall had a single increment of testing near the end. Being part of a team that used agile allowed me to constantly develop and maintain tests in a disciplined manner where cutting corners and finding shortcuts could've been costly for the project. Going forward, research testing methods for language-specific technologies and design patterns before writing tests so that if there is increased complexity within singular methods, the process can be smoother. 




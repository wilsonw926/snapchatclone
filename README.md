# Project 2: Snapchat Clone #
# Part 2: Backend #

## Due Date ##
Tuesday, April 11th at 11:59 pm

## Description ##

**Note: You will need an iPhone / iPad to test this project on. You may work with a partner if you do not have a device to use**

Congrats on making it this far on the Snapchat Clone project! By now, we've learned how to build a feed of images as well as to implement our own camera. The next step is to store the images that users take and allow them to share with others. For this part of the project, we'll be using Firebase's Authentication, Database, and Storage modules, which you can reference at any time here: https://firebase.google.com/docs/ios/setup

This part of the project is particularly interesting because you will all be working off of the same database, which means that you will get to share the pictures you take with your classmates! 

A few ground rules about the pictures you take:
- Please refrain from posting anything inappropriate or offensive (ideally we maintain the spirit of the app and limit pictures to memes, dog spots, and other random things). 
- Do not post pictures of solution code - we will know if you do.
- Try not to spam; there's a lot of students in this class and no one wants to scroll through hundreds of pictures at a time. 

Also very important:
- Because everyone will be writing to the same database, it is EXTREMELY important that you are very careful when it comes to functions that involve writing data. Saving data to an incorrect path or storing data improperly not only affects your own project, but the class as a whole. For that reason, we have created a Constants file where all of the possible Firebase endpoints are listed. When writing data to a particular path, you should only use these constants - **DO NOT USE STRING LITERALS TO SPECIFY A PATH**. If we catch you specifying a path directly as a string and discover any typos on Firebase, you will risk losing points on your project. 

One more quick note:
Almost every function that you need to implement for this project involves the use of closures. If you're not clear on how those work, you should definitely review the lecture slides again before attempting to do anything.

## Downloading the Project ##

For this project, we will be requiring everyone to submit via the GitHub submission method on Gradescope.

Instead of downloading the lab as a zip, you'll need to create a new repository for your changes. You can do this by tapping on "Fork" in the top right of this page. Then open up your terminal, navigate to the directory you want to put your project in (i.e. `cd Desktop`), and clone your repository using the following command (replace YOUR-USERNAME with your github username). 
	
	git clone https://github.com/YOUR-USERNAME/ios-decal-proj2-part2

This will create a repository on your computer that you can commit and push your changes to (it's good practice to do this frequently). When you are done with the project make sure you add all of your files to your repository, and push the changes. You can do this using the following commands in your `ios-decal-proj2-part2` folder (type `cd ios-decal-proj2-part2` into terminal to get into the directory if you are not yet in it)

	git add .
	git commit -m "Finished Project 2-2!"
	git push origin master
	
Once you have done this, you can view the files you pushed at https://github.com/YOUR-USERNAME/ios-decal-proj2-part2. You can then use this repository to submit via Gradescope when you are finished (see the **Submission** section below).

## Getting Started ##

For this part of the project, it's ok if you weren't able to successfully implement the camera or feed - this is taken care of for you. 

You also do not need to handle any of the Firebase installation or setup. Be sure that you're only working off of the SnapchatClonePt3.xcworkspace file (not the xcodeproj file!) since this includes the Firebase cocoapod. 

You'll specifically be editing the following files:

1. `LoginViewController.swift`
2. `SignupViewController.swift`
3. `CurrentUser.swift`
4. `ImageFeed.swift`
5. `PostsTableViewController.swift`

Each of the functions you need to implement is marked with a "TODO" and detailed specs of what needs to be completed. You should be able to follow the specs in the code comments to implement everything. 

## Understanding the Firebase Structure ##
Before you begin to write any code, take a second to understand the hierarchy of the JSON tree, as shown below:

![](/README-images/jsontree.png)

As you can see, we split the data into two larger structures - Users and Posts. Remember that the Users node has nothing to do with authentication (which is taken care of in the authentication module) but only to store additional information about the user which the Auth instance does not support. In this case, we want to store a list of id's of all the posts that the user has opened, to prevent them from being able to open them again. In the image above, the user with uid "J0hq0NXe2ONoSuewOSN6CPFRwWe2" (this is their actual uid, not just a child by auto ID) has read all four of the posts in the Posts node. 

Meanwhile, every post is defined by four essential properties: date, imagePath, thread, and username. Remember that the imagePath corresponds to the location in the storage module of the actual image data. 

## Authentication ##

You'll notice that if you run the app at this point, you should be greeted with a new login screen. Having user accounts allows us to distinguish who took each picture and makes it much easier to handle some of the logic. 

First, open `LoginViewController.swift`, where you will need to implement the function `didAttemptLogin`. This function simply takes the value of the email and password textfields and attempts to log in through Firebase, and finally segues to the main app. 

You'll need to do something similar for `SignupViewController.swift`. Remember, however, that Firebase's createUser function doesn't allow us to set a displayName property. After creating the user, in the completion handler, you'll need to create a profile change request for the returned user and commit its displayName as well. 

## Storing a User's Read Posts ##

Open the file `CurrentUser.swift` in the `Model` folder. You'll see that there are two functions that need to be implemented. 

The `addNewReadPost` function will be called whenever the user clicks on a particular post, and will add the ID of that post as one of the user's readPosts. It may be easier to implement this one first. 

The `getReadPostIDs` function allows us to retrieve all of the ID's of the user's read posts. For reference, this will be called when we attempt to load the tableview with all the posts so that we can identify which ones are clickable and which ones are not. 

## Storing and Retrieving Posts ##

Go to `ImageFeed.swift` now, where you'll find three functions (`addPost`, `store`, and `getPosts`) which are still to be implemented. 

In the `addPost` function, you'll be saving the data for an image every time the user posts to a feed. Remember that a post is defined by a date, imagePath, username, and thread. The latter two are provided as parameters, and the path is already defined for you, but for the date you'll need to create a string corresponding to the current date. Create a dictionary with these parameters and pass that in as the value to a new child node in Posts. Finally, since the database doesn't store the actual image data, you should call the function to store the data separately.

In the `store` function, you will save the actual image data to a path in the storage module. 

Finally, the `getPosts` function will retrieve all of the posts from Firebase as well as the current user's read post ID's, and then create a Post array where each post's read property is defined by checking if the id is contained in the read posts. For this part, you'll need to nest closures together to make sure everything is performed in order. This function may end up being a bit large, but the specs in the comments should clearly describe what to do. 

After completing this part, you should be done with all of the Firebase function calls!

## Connecting it all together ##

The final step is to use the functions we've implemented to load our feed and update it accordingly. 

Open `PostsTableViewController.swift` and take a look at the function `updateData`. Here we'll be setting up the data for our tableview by making a `getPosts` request and storing the results in the thread dictionary we used in part 1. We'll also load the images for each post in the background and store references to them in the dictionary `loadedImagesById` (we can do this in the background because the user wont expect to see the images immediately - only after clicking a cell). 

You'll also need to finish the `viewWillAppear` function as well as the `didSelectRowAt` function at the bottom (which will update a user's read posts when they click on a cell). 

And that's it! If everything works at this point, you should be able to both see your classmates pictures and share your own. You've now made a working clone of Snapchat!

## Grading ##
Once you have finished, please submit your files to [Gradescope](https://gradescope.com/courses/5482). You will need to submit files EVEN if you are being checked off, since Gradescope does not support submission-less grading at the moment. We have enabled group submission for this assignment, so make sure to include your partner's name if you only worked on one computer.

To submit, please upload your code to either GitHub or Bitbucket, and use the "Github" or "Bitbucket" submission feature on Gradescope (we've experienced the fewest amount of bugs with students who have submitted this way). Please check out the [slides in Lecture 3](http://iosdecal.com/Lectures/Lecture3.pdf) for step-by-step submission instructions if you're confused about how to do this (or ask a TA!)

If you are unable to submit via GitHub you can submit your lab as a zip folder (**Note: there will a very good chance we will need to e-mail you asking you to re-submit due to Gradescope zip submission bugs**). To do this please open your ios-decal-proj2-part2 folder, and compress the contents inside (not the folder itself). This should generate a file, **Archive.zip**, that you can submit to Gradescope.

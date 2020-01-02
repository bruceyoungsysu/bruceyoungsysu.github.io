---
title: Sesame Movie Implementation
categories:
 - Spring Boot
 - React.js
tags: React
image: https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/sesame_movie/img_3.png
---

In this post we implemented both the front-end and the back-end of Sesame Movie website utilizing React.js and Spring Boot respectively. The basic functions implemented in this website include browsing movies by trending or genres, browsing game detailed information, and user register or login to record their preferences on each movie. More functions like user creating movie lists, rate or write reviews for a movie.

##Sesame Movie React Server



#### Features

This movie website is developed for rating and archiving the movies for each user. The data of this movie site is from ****the movie database(TMDB)****. The features implemented in this site include:

1. **Movie Browsing**

  On the front page of the Sesame Movie, I implemented two decks of movies in order to display movies in the form of cards. Each card is comprised of a fixed-size poster, movie's title and its rating from TMDB. There is also a group of forwarding and backward arrows to help the user to navigate to a different group of cards. 

   ![](https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/sesame_movie/img_1.png){:width="400px"}

  When the cursor is hovering over the poster, there will be a popup window showing the detailed introduction of the corresponding movie including the movie's full name, publishing date and content of the movie.

   <img src="https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/sesame_movie/image-20191229171633864.png" alt="image-20191229171633864" width=400px/>

  The movie genres deck also provides tags to navigate to each genre by clicking on the corresponding text

   <img src="https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/sesame_movie/img_3.png" alt="image-20191230141836979" width=700px align="center"/>

2. **Movie Details**

  By clicking on each movie's poster, the user will be able to navigate to the detail page of each movie. The detailed information of a movie includes a high-resolution poster of the movie, the movie name and its publishing date, a detailed overview and the genres it belongs to. There is also a background image crossing the whole width of the page.

   <img src="https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/sesame_movie/image-20191230143008418.png" alt="image-20191230143008418" width=600px align="center"/>

  Below the detailed information of the movie, there lists the top-billed casts and top reviews in the form of decks

   <img src="https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/sesame_movie/image-20191230144439731.png" alt="image-20191230144439731" width=700px/>

   <img src="https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/sesame_movie/image-20191230144547827.png" alt="image-20191230144547827" width=700px />

3. **User Login and Like a Movie**

  The website also supports users to register or login. After logging in, the registered user name will be shown instead of the login button. 

  The login popup is shown as follows:

   <img src="https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/sesame_movie/image-20191230145943707.png" alt="image-20191230145943707" width=300px/>

  The header after login when we are using the user name of **react**:

   <img src="https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/sesame_movie/image-20191230153607367.png" alt="image-20191230153607367" />

  After logging in, the user can keep their own record of liked movies by clicking on the heart icon. If a movie is already in the list of favorite movies, the heart icon will be shown as red:

   <img src="https://raw.githubusercontent.com/bruceyoungsysu/bruceyoungsysu.github.io/master/_posts/sesame_movie/image-20191230162803175.png" alt="image-20191230162803175" width=500px align="center"/>

  By clicking on the red heart again, the user can remove the movie from the favorite list.



####Implementations

The front end server is implemented in React.js. Its components mainly fall into the following several categories:

- The content page: Contains the elements in the main page
  - Header.js: The website logo, search bar and login buttons.
  - BriefToolTip.js: The popup tooltip when hovering curser on the movie poster.
  - MovieCard.js and MovieCards.js: movie card and movie card decks.
  - MovieGenres.js and MovieTrending.js: movie genres deck, movie trending deck and their navigation logics.

- The detailed page: Contains the detail information of a movie
  - CastCrews.js and CastDetail.js: top billing casts card and deck
  - MovieDetail.js and DetailInfo.js: detailed information of a movie, including the banner background image, poster, movie genres and introduction
  - MovieButtons.js: Four buttons for users to take any action on a movie, currently only the function of **like a movie** is implemented
  - Reviews.js and ReviewDetail.js: Review cards and the review deck holding the reviews
  - Login: The logic of user registration, login and log off

- LoginBar.js: the appearance of the login and register buttons, depending on the status of whether user has logged in
  - LoginPopup.js: the appearance and logic of the login popup window
  - RegisterPopup.js: the appearance and logic of the register popup window

## Sesame Movie Java Server

#### Overview

In this repository, we implement the Java server of the [Sesame Movie](https://sesame-movie.herokuapp.com/). The Java server handles the communication between the relational database and the front end utilizing the MVC architecture. In the model module, we implement the class of each entity by specifying their constructor and setters and getters. It forms a sense of what information a class will take. Meanwhile, the crud repository handler of each entity is created correspondingly which simply extends the CrudRepository from the spring framework.

Both the entities and its curd repos are called in the service handlers. In each service handler, the `@RestController` annotation is used to create restful API for the front end. @Autowired annotation is used to link the service and repo. The logic of methods each post and get requests are calling are implemented in the service module of each entity annotated with corresponding get or post method. 

In the implementation of some methods, cookies are utilized to keep track of the user's session. In each session, we put the User class into a HttpSessionattribute curuser to authenticate the user who is sending HTTP requests.



#### Implementation

The logic of back end service of Sesame Movie can be separated into the following entities/classes:



**The movie entity**: holds the information of a movie and this information can be used to formalize a movie card. The information of a movie includes the title, production date, poster, introduction, casts, reviews, etc. All of the information is requested directly from the movie database API thus no entity and service are implemented to handle the requests. Each movie is identified with a unique movie id in the API.



**The user entity**: The user entity holds the information to identify a user including the user name, password and email address. When registering, the user needs to provide this information. The user service module provides the functions to register or login as a specific user, getting the profile of a user and logout the session of the current user. Each user is identified with a generated user id.



**The like entity**: The like entity holds the information about a certain user likes a certain movie. It uses the user_id to identify each user and movie_id to identify each movie. Also, it utilizes a boolean variable to note whether this user likes this movie. If a record does not exist in the database, it means the user has not provided the preference over this movie which means the user does not like it by default.



#### Notes

In each service module we handled the cors issue by setting the following annotation:

```java
@CrossOrigin(origins="https://sesame-movie.herokuapp.com", allowCredentials = "true", allowedHeaders = "*")
```

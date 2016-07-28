# Learning by Doing 

![Ubuntu version](https://img.shields.io/badge/Ubuntu-16.04%20LTS-orange.svg)
![Rails version](https://img.shields.io/badge/Rails-v5.0.0-blue.svg)
![Ruby version](https://img.shields.io/badge/Ruby-v2.3.1p112-red.svg)

Mackenzie Child's video really inspired me. So I decided to follow all of his rails video tutorial to learn how to build a web app. Through the video, I would try to build the web app by my self and record the courses step by step in text to facilitate the review.


# Project 4: How To Build A Pinterest Clone With Rails

This time we're going to build a Pinterest like application which includes the pins, users, image uploading, and voting as well. A user must sign-in in order to  submit a new pin or vote, as well as to delete a pin. And then we got some amazingly going on, so the pins will scale to the size of the PinBoard(window size).

https://mackenziechild.me/12-in-12/4/


### Highlights of this course
1. Users
2. Pins
3. Image Uploading
4. Voting
5. HAML
6. Masonry
7. Bootstrap 

# Create a PinBoard
```console
$ rails new pin_board
```

Chage directory to the pin_board. Under `pin_board/Gemfile`, add `gem 'therubyracer'`, save and run `bundle install`.      

Note: 
Because there is no Javascript interpreter for Rails on Ubuntu Operation System, we have to install `Node.js` or `therubyracer` to get the Javascript interpreter.

```console
$ bundle install
```

Then run the `rails server` and go to `http://localhost:3000` to make sure everything is correct.


To be continued...
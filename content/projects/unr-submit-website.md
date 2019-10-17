---
title: "UNR Submit Website"
date: 2015-05-05T12:00:00-07:00
draft: false
---

[Submit on GitHub](https://github.com/Naosyth/SubmitWebsite)

## About This Project

This was my senior project for college, made in a group of three along with [Nolan Burfield](https://www.linkedin.com/in/nolan-burfield-0883866b/) and [Hardy Thrower](https://www.linkedin.com/in/hardy-thrower/). It was developed under the guidance of [Dr. Fred Harris](https://www.unr.edu/cse/people/fred-harris) for use with some of his introductory computer science classes. The web app was built to replace an existing web app being used that allowed students to create accounts and upload programs written in C/C++ and automatically compile them and run them against a suite of tests. The application had several roles for students, teaching assistants, and professors. Once a program had been submitted and the assignment deadline reached, teaching assistants could view the submitted program directly from the web app and leave comments for students to review.

## Technology

This was a fairly standard Ruby on Rails application. Student code was compiled via a system call, which is obviously a security vulnerability. For the initial development, this was deemed good enough, since the existing application also compiled student code this way. Further development could have perhaps spun up virtualized environments to compile student code on more safely.

## Demo

{{< youtube yeBVpiUDwgE >}}

---
title: "PUBGderboards"
date: 2019-01-14T12:00:00-07:00
draft: false
---

[PUBGderboards on Github](https://github.com/timendez/pubg-leaderboards)
[PUBGderboards.com](https://www.pubgderboards.com/)

## About This Project

PUBGderboards is a leaderboard website for the popular battle royale videogame [PlayerUnknown's Battle Grounds](https://www.pubg.com/). PUBGderboards is one of PUBG's [featured apps](https://developer.pubg.com/featured_apps?locale=en). It allows players to see how they compare to other players from multiple platforms, either on a per-season basis or aggregated across all recorded seasons. At the time of its release, PUBGderboards was the only app that had the ability to combine stats from all seasons, since this was not yet built in to the PUBG developer API. I created the front-end for the application, while my friend [Tim](https://github.com/timendez) built the back-end.

## Technology

PUBGderboards is a React app built with Webpack 4, Babel 7, and Stylus. One of my main goals with this project was to get familiar with new changes in Webpack 4. Although the scope of this application was fairly narrow - just showing player stats, it had some interesting design and UI/UX challenges to overcome. We wanted these leaderboards to be very easy to share with friends, so we made sure players could easily copy/paste leaderboard URLs with each other. We also wanted to allow users from different platforms to directly compare their stats to one another, so we had to implement platform selection on a per-user basis rather than a per-leaderboard basis.

In addition to having certain features and user-flows in mind, we wanted to keep with the general PUBG design themes, so we stuck to the official PUBG colors and fonts.

## Screenshots

{{< figure src="https://i.imgur.com/sVc3Idv.png" width="750px" title="A comparison of Shroud's stream snipers" link="https://i.imgur.com/sVc3Idv.png" >}}

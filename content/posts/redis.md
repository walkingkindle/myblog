+++
date = '2025-05-21T12:38:52+02:00'
draft = true
title = 'What I learned implementing my own Redis'
+++


About a month ago, I set out on a journey to try and create my own redis implementation. Being the lazy bastard that I am, I opted out for a convenient "gamified" solution, like [CodeCrafters](https://codecrafters.io) where some of the steps are outlined for you and you just need to make your implementation and push to your GitHub, where the pipeline will run the tests for this particular step, after which you should refactor your code to implement the solution.

Many times for a project like this, looking at this from "afar" you think, "It's just a key-value store, right? How hard can it be?"

Boy was I wrong.

The project lets you really start from scratch, with basic TCP packages are the only thing that is allowed, which should receive and store the keys based on a [particular key structure/file format](https://rdb.fnordig.de/file_format.html), which should stay offline indefinately. There are many extensions that are supported as addons. Overall it's a great coding exercise that can sharpen your skills with a project based learning whilst you learn how some of the fundamental elements of programming work, that we use every day.
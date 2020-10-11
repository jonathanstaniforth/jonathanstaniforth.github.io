---
layout: post
title:  "Laravel with Docker"
date:   2019-10-05 19:00:00 +0800
tags: [docker,laravel,git]
navigation: true
---
It has been a long time since I have done backend development, as recently I have been focusing on server configuration and frontend work. However, a new personal project that I have been working on has got me involved with backend dev again. This project started when forest fires in Indonesia created a haze that affected several countries in South East Asia. Many people were affected, including myself, and I started to read more into the issue, eventually coming across an idea for a new project.

The project would require a simple system at first, that would later expand to include more complex additions. For the backend of the system, I decided to go with a PHP framework called [Laravel](https://laravel.com). I have used this framework in the past and like how it allows for the development of simple, and elegant systems.

Laravel offers several virtual environments for development purposes, but, I wanted to use Docker so that I could easily transfer the environment to a server. However, after searching for a while, I could not find an appropriate Docker image. Therefore, I created one, seizing the opportunity to use the [GitHub Package Registy](https://help.github.com/en/articles/about-github-package-registry#about-github-package-registry) to host the image.

The GitHub repository for the Laravel Docker image can be found [here](https://github.com/jonathanstaniforth/laravel-docker), and contains instructions on how to use the image. Navigate to the **package** tab to find the image itself. Additionally, I created a Docker Compose file to quickly setup the Laravel Docker image with a MySQL database, which can be found [here](https://gist.github.com/jonathanstaniforth/e03cfd0a91566d382f300d3853b63ffb) as a GitHub Gist. Later, I will expand this to include a PostgreSQL database option as well.

## GitHub Package Registry
A quick review of the GitHub Package Registry. Overall it does not offer many features, simply holding Docker images or other packages, but, it works solidly. It is nicely coupled with the GitHub repository, allowing for the code and package to be together, rather than split over different services.

One comment is that it seems GitHub is directing Docker image tags toward semantic versioning, moving away from the controversial latest tag, which [Docker Hub](https://www.docker.com/blog/introducing-docker-hub-improved-tag-ux/) has been working on as well.

Over time, it would be nice to see features such as auto-build, and references to particular commits being added.

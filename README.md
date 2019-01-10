# TakeHomeTest
KoreConX Take Home Test 

***copyright***     Copyright 2018 KoreConX Inc. 

 ***link***          http://www.koreconx.com

 ***package***       KoreConX

 ***since***         KoreConX v2.0.0

 ***author***        KoreConX Inc.


## How to make it run?

1. Clone the repo
1. Run composer install && npm install (you need to install composer and node BTW)
1. Ccreate a new database
1. Create the .env file and modify DB connection (you can copy the .env.example)
1. Run the command ./artisan jwt:secret (the api routes are secured with jwt)
1. Run the command ./artisan key:generate (or set the APP_KEY manually in the .env file)
1. You can set a virtualhost or use the ./artisan serve

### what to expect?
After registering a new user you'll be automatically logged-in, a home page will show 2 options Vuejs version or Laravel Version both look for the same goal but with different approaches, for the purposes of simplicity I've made different controllers for both versions.


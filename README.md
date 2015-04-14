App Engine application for the Udacity training course.

## Products
- [App Engine][1]

## Language
- [Python][2]

## APIs
- [Google Cloud Endpoints][3]

## Setup Instructions
1. Update the value of `application` in `app.yaml` to the app ID you
   have registered in the App Engine admin console and would like to use to host
   your instance of this sample.
1. Update the values at the top of `settings.py` to
   reflect the respective client IDs you have registered in the
   [Developer Console][4].
1. Update the value of CLIENT_ID in `static/js/app.js` to the Web client ID
1. (Optional) Mark the configuration files as unchanged as follows:
   `$ git update-index --assume-unchanged app.yaml settings.py static/js/app.js`
1. Run the app with the devserver using `dev_appserver.py DIR`, and ensure it's running by visiting your local server's address (by default [localhost:8080][5].)
1. (Optional) Generate your client library(ies) with [the endpoints tool][6].
1. Deploy your application.


[1]: https://developers.google.com/appengine
[2]: http://python.org
[3]: https://developers.google.com/appengine/docs/python/endpoints/
[4]: https://console.developers.google.com/
[5]: https://localhost:8080/
[6]: https://developers.google.com/appengine/docs/python/endpoints/endpoints_tool



## Task 1:
### Design:
- I choosed to not use inheritance for Session as the datastore allows, because I prefer to explicitly define a property ('conference') to define the relation. It sounds to me more logical and readable.

### Endpoints:
- `sessions/create/{ConferenceKey}` > `conference.createSession`
- `sessions/{ConferenceKey}` > `conference.getConferenceSessions`
- `sessions/by/speaker/{speakerName}` > `conference.getSessionsBySpeaker`
- `sessions/{ConferenceKey}/type/{sessionType}` > `conference.getConferenceSessionsByType`

## Task 2:
### Design:
- I choosed to add the wishlist as a property ('sessionKeysWishlist') in the Profile model, as I think is the most efficient way to do it.

### Endpoints:
- `wishlist/add/{websafeSessionKey}` > `conference.addSessionToWishlist`
- `wishlist/get` > `conference.getSessionsInWishlist`
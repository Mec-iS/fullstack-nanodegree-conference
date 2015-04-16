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
- The SessionKey can be retrieved via `conference.getConferenceSessions`

### Endpoints:
- `wishlist/add/{websafeSessionKey}` > `conference.addSessionToWishlist`
- `wishlist/get` > `conference.getSessionsInWishlist`

## Task 3:
- I added the index needed by the queries for Session objects in the index.yaml as explained in the file and in the documentation:
```
- kind: Session
  properties:
  - name: conference

- kind: Session
  properties:
  - name: speaker
```

- Additional Queries:
I have used this [cheatsheet](https://docs.google.com/document/d/1AefylbadN456_Z7BZOpZEXDq8cR8LYu7QgI7bt5V0Iw/edit) to have a general look on the NDB.

-- Most of the needs about querying on Conference objects are satisfied in the app by the `conference.queryConferences` method that allows the user to build her/his own query via a JSON defining multiple filters into a POST request, like:
<pre>
{
  "filters":
    [
      {
        "field":"MONTH"
        "operator":"="
        "value":"4"
      }
    ]
  }
</pre>
With a request body like this (asking for all the conference in the month of April), any user can query directly the datastore with any filter(s) she/he wants.

-- Query 1: On the Session objects it could be useful to have a query on a particular highlight for a particular Conference:
```
...
highlight = request.highlight
sessions1 = Session.query(Session.conference == ndb.Key(urlsafe=request.conferenceKey)).filter(Session.highlights == highlight)
...
```

-- Query 2: Another query can be one that let the user to get all the sessions in the same day of a Conference, ordered by startTime
```
...
date = datetime.strptime('2015-4-24', "%Y-%m-%d").date()
sessions = Session.query(Session.conference == ndb.Key(urlsafe=request.conferenceKey)).filter(Session.startDate == date).order(Session.startTime)
...
       
```
Implementations endpoints:<br>

`sessions/{websafeConferenceKey}/by/highlights/{highlight}` > `conference.getConferenceSessionsByHighlight`
`sessions/{websafeConferenceKey}/by/date/{conferenceDate}` > `conference.getConferenceSessionsByDate`


- Query Problem:Probably the problem is that: `Only one inequality filter per query is supported`.<br>
The datastore doesn't allow to have two inequality filters in the same query: 
```
sessions = Session.query(Session.typeOfSession != 'workshop', 
                          Session.startTime < datetime.strptime('19:00', '%H:%M').time())  # IT DOESN'T WORK
```
Returns:
```                 
BadRequestError: Only one inequality filter per query is supported. Encountered both typeOfSession and startTime
```

Probably the easiest way is to apply a filter via query and then iterate the results to find the objects that match the second filter using list comprehension:<br>
```
sessions = Session.query(Session.typeOfSession != 'workshop')
results = [s for s in sessions if s.startTime < datetime.strptime('19:00', '%H:%M').time()]
```

Otherwise it's possible to run two separate queries and then look with some scripting at the intersection of this two query sets, as suggested [here](http://stackoverflow.com/a/24875358).


## Task 4

### Design:
I implemented Memcache for featured speakers at line 567 in `conference.py`, inside `_createSessionObject()`. After a session is put in the datastore the script checks if the speaker has more than one session in that conference.
As a key for Memcache I choosed a string like `{websafeConferenceKey}:featured`, to specify that is referred to featured speakers of a given Conference.

## Endpoint:
`conference/{websafeConferenceKey}/featuredSpeaker` > `conference.getFeaturedSpeaker`

## Comment:
The endpoint serves exclusively from the cache.<br>
I created a custom message class to handle the message (`model.FeaturedSpeakerMessage`) and a custom form class (`model.FeaturedSpeakerForm`), so the endopoint returns a structure with the Conference key and a 'featured' property
containing the Conference key and the list of featured speakers with relative sessions.

There could be a problem of memcache expiration, because it could be possible that featured speakers are not loaded in memcache if any user created a Session recently; 
I try a workaround with setting a quite long period of expiration. As far as I know with my experience with GAE It can be solved by setting a 
proper expiration time, based on observations about the loads of the app (e.g. empirically, how often the cache get set usually during production time). 
Or a cron job that periodically add to the cache the  `{websafeConferenceKey}:featured` key with proper values taken from the datastore.
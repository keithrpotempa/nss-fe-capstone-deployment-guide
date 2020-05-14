# NSS FE Capstone Deployment Guide

## TODO: Helpful References to Work in
- https://css-tricks.com/intro-firebase-react/#article-header-id-4

## Firebase
I used firebase because it seemed like the best route for getting the whole deployment package in one place: database hosting, web hosting, and user authentication. 

## Database
What database do I use?
Realtime Database:  it is JSON-like, so you can use your old fetch / api calls without a ton of changes,

You can import your existing JSON file directly using the web console for Realtime Database. 

Note: your API key is meant to be public. You may get a warning or alert from GitHub that your API key is public after you push a commit, but that is the intended behavior. See [link](https://stackoverflow.com/questions/37482366/is-it-safe-to-expose-firebase-apikey-to-the-public) for more info on this. 

## DB Changes
- Realtime Database calls their ids keys. But they don't quite behave the same way as they do in JSON server. 
- Posting a new object will create a key, which is a unique alphanumeric: (like -M6uTZMy41ZsLaFO5p2c) rather than just an integer. So if  you're doing a lot of parseInt on your ids, be ready to work out some issues there. 
- The database shifts one level "deeper" in a way that causes less problems than it might seem. Basically, all the properties for a single object are nested within its id (which is called a "key" in Realtime Database), rather than having the id at the same level. 
- If you plan on keeping ids the same from your JSON server, note that their numbering starts at 0 rather than one, so you may end up with a mismatch between your javascript "id" property and Realtime Database keys... 
- no expand/embed option -- you will have to rework anything that uses them. But you can use 

## API Calls
- In order to make an api call at all, you will need to modify your rules (in the Firebase web console for the project) so that read and/or write are set to true.

Here's a working API call formula to get one object:
- `https://${project-name}.firebaseio.com/${dataType}/${id}/.json`
as in: 
- `https://fate-character-codex.firebaseio.com/characters/-M6uTZMy41ZsLaFO5p2c/.json`
where: 
- `characters` is the datatype I'm retrieving, 
- `-M6uTZMy41ZsLaFO5p2c` is the key (id) of the particular object I'm retrieving
- note `.json` is required at the end to make the response a json object

Unfortunately, Realtime Database has a limited set of parameters it will accept. So you cannot use expand or embed in your calls. Be prepared to have to rework the logic behind any of your code that uses either of these. See here for more details on accepted parameters. 

I have been able to make secondary HTTP GET requests to get expand/embed-like information using indexon and orderby.    

References for indexon and orderby:
- [1](https://firebase.google.com/docs/database/security/indexing-data) 
- [2](https://firebase.google.com/docs/database/rest/retrieve-data#section-rest-filtering)

Here's a working API call formula to get many objects connected by a single foreign key
`https://${projectname}.firebaseio.com/${dataType}/.json?orderBy="${foreignKeyName}"&equalTo="${foreignKey}"`
as in:
`https://fate-character-codex.firebaseio.com/characterAspects/.json?orderBy="characterId"&equalTo="-M6uTZMy41ZsLaFO5p2c"`
where a call for me to get all the aspects of a single character would be:
- `characterAspects` is my join table
- `characterId` is the name of the foreign key of character, the data type I'm trying to filter by 
- and `-M6uTZMy41ZsLaFO5p2c` is the foreign key itself

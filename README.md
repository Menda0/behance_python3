behance_python
==============

A Python wrapper for the Behance API
------------------------------------

####Warning! This wrapper is very much still in development and could change substantially!

Please see [Behance API documentation](http://www.behance.net/dev) to get an API key and more information.

**Note**: Many Behance API endpoints provide data in pages of 12 items. You will
need to page through results until you receive a 500 error (in the form of
a BehanceException) which will indicate that there are no more pages.

#Installation
    pip install behance_python

Or:

    git clone git://github.com/aravenel/behance_python.git

#Usage
All wrapper functionality flows from the main API object which must be
instantiated using your Behance-provided API Key.

All attributes can be accessed using either objects (object.key) or dict 
(object['key']) notation. **Beware!** Some of the JSON returned may have numerical
keys, which Python cannot use for object notation--you will need to access these 
by their dict notation. Additionally, as JSON returns unicode, integer keys have
been converted from unicode to ints to make life easier.

##API Object Usage
```python
from behance_python.api import API

behance = API('your_api_key_here')
```

##Project functionality
###Search for projects
```python
projects = behance.project_search('term1', 'term2', filter_key='filter_value')
project_owner_name = projects[0].owners[129052].first_name
```

Supports all filters and modifiers as supported by Behance.

Data will be returned as list of objects with same keys as Behance API. To save 
on API calls, these are not full Project objects (i.e. they do not have all of 
the attributes you would get from get_project())--you must call the 
API.get_project(project_id) method to get project details including images.

###Get Single Project Details
```python
proj = behance.get_project(project_id)
project_images = [module.src for module in proj.modules if module.type=='image']
```

Returns an instance of the Project object. This object has attributes named
identically to attributes as returned by Behance API. As with the API, 
artwork associated with a project are stored in Project.modules, which is a list
of objects, each object representing one module and its corresponding
metadata.

###Get Project Comments
```python
comments = proj.get_comments()
comment[0].user
```
Method of the Project object. Returns list of objects, each object 
representing a single comment and its metadata.

##User functionality
###Search for Users
```python
users = behance.user_search('term1', 'term2', filter_key='filter_value')
users[0].first_name
>>> Matias
```
Works just like project_search.

###Get Single User Details
```python
user = behance.get_user(user_id_or_username)
```
Returns User object. This object has attributes named identically to attributes
as returned by Behance API. 


###Get User Projects
```python
user_projects = user.get_projects(filter_key='filter_value')
```
Method of the User object. Can optionally include any filters supported by Behance API.
So as not to chew up API calls, these are not actual Project objects. To get 
artwork associated with these projects, you will need to call the API.get_project(project_id)
method.

###Get User Works in Progress
```python
user_wips = user.get_wips(filter_key='filter_value')
```
Method of the User object. Can optionally include any filters supported by Behance API.
So as not to chew up API calls, these are not actual WIP objects. To get 
artwork associated with these projects, you will need to call the API.get_WIP(wip_id)
method.

###Get User Appreciations
```python
user_appreciations = user.get_appreciations(filter_key='filter_value')
```
Method of the User object. Can optionally include any filters supported by Behance API.

###Get User Collections
```python
user_collections = user.get_collections(filter_key='filter_value')
```
Method of the User object. Can optionally include any filters supported by Behance API.

##Work in Progress Functionality
###Search for Works in Progress
```python
wips = behance.wip_search('term1', 'term2', filter_key='filter_value')
```
Works just like project_search.

###Get Work in Progress
```python
wip = behance.get_wip(wip_id)
```
Returns WIP object. This object has attributes named identically to attributes
as returned by Behance API. 

###Get Revision
```python
rev = wip.get_revision(revision_id)
```
Works in progress store multiple revisions. Fetch individidual revisions with
a revision ID. Method of the WIP object. 

###Get Comments
```python
comments = wip.get_revision_comments(revision_id, filter_key='filter_value')
```
Comments are stored associated with a revision. Method of the WIP object. Can optionally
include any filters supported by Behance API.

##Collection Functionality
###Search for Collections
```python
collections = behance.collection_search('term1', 'term2', filter_key='filter_value')
```
Works just like project_search.

###Get Collection
```python
collection = behance.get_collection(collection_id)
```
Returns Collection object. This object has attributes named identically to attributes
as returned by Behance API.

###Get Collection Projects
```python
projects = collection.get_projects(filter_key='filter_value')
```
Returns projects that are members of a collection. Note that these are not actual
Project instances to save on API calls--to get artwork for these, you would need to
call API.get_project(project_id). Can optionally include any filters supported by 
Behance API.


#Exceptions
Unfortunately, they happen. If an exception happens in the calling of the API
but before a response is returned (e.g. a timeout), the library will raise 
the exception from the underlying Requests library. If the response from the 
Behance API is anything other than status code 200, it will raise an exception 
corresponding to the error code. These exceptions are all subclasses of the 
BehanceException, and they are:
- Forbidden (403 error)
- NotFound (404 error)
- TooManyRequests (423 error)
- InternalServerError (500 error)
- ServiceUnavailable (503 error)

If any other error code is received, will throw a generic BehanceException with
the actual error code stored in attribute self.error_code.

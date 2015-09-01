---
layout: post
title: Creating a dynamically updating search field using ui-select!
---

I was recently working on project utilizing the Spotify api in order to search spotify’s library and select a song. For this project, my team and I used the MEAN stack. Obviously, in order for the user to search Spotify’s library, we needed to have a search field on the front end. However, angular does not have any native means to create a dropdown menu that updates with suggested options. This creates a problem since the user never receives feedback from the search field and therefore is cannot be sure whether their search is actually returning valid data. Additionally if the user is not getting a list of suggested options then they are required to fully type their desired option in order to select a song (which is inconvenient).

After doing some research, I discovered an angular plugin called ui-select. ui-select abstracts away the code needed to build a functional search drop-down.  

For the purpose of this example I'm going to work backwards.

First I created a request handler on the server which returns a list of all users in the database when called.

```JavaScript

findAllUsers: function(req, res) {
  User.find({}, function(err, docs) {
    if (!err) {
      res.json(docs);
    } else {
      console.error(err);
    }
  });
}
  ```
  
Then I created a factory in order to send user intent to the back end of the application. The first function in the factory calls the findAllUsers function and passes the data back to the controller that called it. For the purpose of this example I have left off the request handler for addFriend but it simply adds a selected user (from the search field) to the currently logged in user's friends list. 
  
  ```Javascript
.factory('userFactory', function ($window, $http) {

var findUsers = function () {
  return $http({
    method: 'GET',
    url: 'api/users/tabs'
  })
  .then(function (resp) {
    return resp.data;
  });
};
var addFriend = function(user){
  return $http({
    method: 'POST',
    url: '/api/users/tabs',
    data: {token: $window.localStorage.getItem('com.beer-tab'), _id: user._id}
  })
    .then(function (resp) {
      console.log(resp.data);
        return resp.data;
  });
}

  return {
    addFriend:addFriend,
    findUsers: findUsers
  };
})
```
The next step is to create the controllers. The findUser controller calls the findUser factory function and then filters through the returned data (an array of user objects). The findUser controller pushes any of the objects whose first name or last name property matches what the user has typed into the search field, into the results array. AddUSer passes the selected user object from ui-select field to the addFriend factory function so it can be added to the database.

```javascript
$scope.findUser = function (inputStr) {
  $scope.results = [];
  userFactory.findUsers().then(function (data) {
    if (inputStr.length > 0) {
      for (var i = 0; i < data.length; i++) {
        if (data[i].name.first.toLowerCase().match(inputStr.toLowerCase()) !== null ||
        data[i].name.last.toLowerCase().match(inputStr.toLowerCase()) !== null) {
          $scope.results.push(
          {name: data[i].name.first + ' ' + data[i].name.last, username: data[i].username, _id:
          data[i]._id});
        }
      }
    }
  });
};

$scope.addUser = function(user){
  if(user){

    if(AuthService.isAuth()){
      userFactory.addFriend(user)
      .then(function(data){
        console.log('CALLED FROM CALLBACK', data)
        $scope.network.push(data);
      })

    }
  }
};

```
Now we're finlly at the HTML and the ui-select. There is a lot going on this snippet of code but the crux of it is the ui-select-choices line. In this line we call findUser on whatever text is being entered by the user. The results that are created by the call to findUser are then displayed in divs that follow the template contained in the ng-bind-html. If one of those divs is selected, the addUser function is called for the user represented by that div.

```html
<ui-select ng-model='result.selected' on-select='addUser(result.selected)'  
theme="bootstrap" ng-disabled="false" reset-search-input="true" 
uiSelectConfig.appendToBody = true; style="width: 300px;">
  <ui-select-match id='ui-select-container' placeholder="I owe a beer to...">
  {{$select.selected.name}}
  </ui-select-match>
  <ui-select-choices refresh='findUser($select.search)' 
  repeat='user in results|limitTo:10'  refresh-delay='0'>
    <div ng-bind-html='user.name | highlight: $select.search'></div>
  </ui-select-choices>
</ui-select>
```

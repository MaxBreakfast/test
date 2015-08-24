---
layout: post
title: Creating a dynamically updating search field using ui-select!
---

I was recently working on project utilizing the Spotify api in order to search spotify’s library and select a song. For this project, my team and I used the MEAN stack. Obviously, in order for the user to search Spotify’s library, we needed to have a search field on the front end. However, angular does not have any native means to create a dropdown menu that updates with suggested options. This creates a problem since the user never receives feedback from the search field and therefore is cannot be sure whether their search is actually returning valid data. Additionally if the user is not getting a list of suggested options then they are required to fully type their desired option in order to select a song (which is inconvenient).

After doing some research, I discovered an angular plugin called ui-select. ui-select abstracts away the code needed to build a functional search drop-down.  


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
  ```Javascript
  .factory('beerPmt', function ($window, $http) {

  var findUsers = function () {
    return $http({
      method: 'GET',
      url: 'api/users/tabs'
    })
    .then(function (resp) {
      return resp.data;
    });
  };
  var addToNetwork = function(user){
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
    addToNetwork:addToNetwork,
    findUsers: findUsers
  };
})
```
```javascript
  $scope.addUser = function(user){
    if(user){

      if(AuthService.isAuth()){
        beerPmt.addToNetwork(user)
        .then(function(data){
          console.log('CALLED FROM CALLBACK', data)
          $scope.network.push(data);
        })

      }
    }
  };

  $scope.findUser = function (inputStr) {
    $scope.results = [];
    beerPmt.findUsers().then(function (data) {
      if (inputStr.length > 0) {
        for (var i = 0; i < data.length; i++) {
          if (data[i].name.first.toLowerCase().match(inputStr.toLowerCase()) !== null || data[i].name.last.toLowerCase().match(inputStr.toLowerCase()) !== null) {
            $scope.results.push({name: data[i].name.first + ' ' + data[i].name.last, username: data[i].username, _id: data[i]._id});
          }
        }
      }
    });
  };
```

```html
           <ui-select ng-model='result.selected' on-select='addUser(result.selected); toggle()'  theme="bootstrap" ng-disabled="false" reset-search-input="false" uiSelectConfig.appendToBody = true; style="width: 300px;">
              <ui-select-match id='ui-select-container' placeholder="I owe a beer to...">{{$select.selected.name}}</ui-select-match>
              <ui-select-choices repeat='user in results|limitTo:10' refresh='findUser($select.search)' refresh-delay='0'>
                <div ng-bind-html='user.name | highlight: $select.search'></div>
              </ui-select-choices>
            </ui=select>
```

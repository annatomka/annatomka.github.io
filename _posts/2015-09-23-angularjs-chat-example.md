---
layout: post
title: AngularJS Chat
---
This is an example that shows how to create a simple Chat application with the following technologies: AngularJS, Angular Material, Socket IO.

The full example is available on branch *master*, but you can start from nearly scratch when you checkout branch *step_registration*.

### Demo
Demo is available here: https://angularchatexample.firebaseapp.com

![](/public/angularchat_2015_11_03.png)

### Main features:
* Login
* Registration
* Logout
* View/Update Profile
* Create room
* List rooms
* Open room
* List users in a room
* List messages in a room
* Create new message

## Source

##### Git
If you use Git, you can clone the repository of the application with the following command:
{% highlight js %}
git clone https://github.com/annatomka/angular-chat.git
{% endhighlight %}

### Download as zip
If you don't want / can't use Git, you can download the source from the following url:
{% highlight js %}
https://github.com/annatomka/angular-chat/archive/step_registration.zip
{% endhighlight %}
### Registration

We have a login form as a starting form, but unfortunately there's no user we could log in to the chat. For this purpose let's create the registration form in this section. Extend index.routes.js with the following route:
{% highlight js %}
.state('registration', {
url: '/registration',
templateUrl: 'app/registration/registration.html',
controller: 'RegistrationController',
controllerAs: 'registrationCtrl'
})
{% endhighlight %}

As you can see from the code above we need a template and a controller too... Create folder *registration* and place registration.controller.js here with the following content:
{% highlight js %}
(function () {
'use strict';

angular
  .module('angularChat')
  .controller('RegistrationController', RegistrationController);

/** @ngInject */
function RegistrationController($rootScope, $scope, UserService, User, AccountService, $state) {
  var registrationCtrl = this;
  registrationCtrl.user = new User();

  registrationCtrl.register = function () {
    UserService.create(registrationCtrl.user).then(function (user) {
      $rootScope.toast("Registration successful! You are now logged in :)");
      AccountService.login(user.username, user.password).then(function(){
        $state.go("rooms",{},{reload: true});
      });
    }, function (error) {

    });
  }
}
})();
{% endhighlight %}

Add registration.html with the following HTML:
{% highlight html %}
<div class="registration-frame" ui-view="" flex="" layout="row">
<div layout="row" flex="" layout-padding="" layout-fill="" layout-align="center center" class="ng-scope">
  <div flex="40" flex-lg="50" flex-md="70" flex-sm="100">
    <md-card class="md-cyan-theme">
      <md-toolbar class="md-cyan-theme" layout-padding>
        <div layout="column" layout-align="center" class="padding-20 text-center">
          <img ng-src="img/logo.png">
          <h1 layout="row" layout-align="center center" class="md-headline ng-scope" translate="">Registration</h1>
        </div>
      </md-toolbar>
      <md-content class="md-padding md-cyan-theme">
        <form name="registration">
          <basic-info user="registrationCtrl.user"></basic-info>
           <md-dialog-actions layout="column" layout-align="center">
            <md-button class="md-raised md-primary" layout="row" layout-align="center center" ng-disabled="registration.$invalid" ng-click="registrationCtrl.register()">
              Registration
            </md-button>
            <a
              class="md-primary md-button md-cyan-theme" ui-sref="login"
              aria-label="Already registered? Login now">Already registered? Login now</a>
             </md-dialog-actions>

        </form>
      </md-content>
    </md-card>
  </div>
</div>
</div>
{% endhighlight %}

We need to link somehow the registration form with the login form, so add a registration button to login.html as the following example shows:

{% highlight html %}
<a
class="md-primary md-button md-cyan-theme" ui-sref="registration"
aria-label="Don't have an account? Create one now">Don't have an account? Create one now</a>
{% endhighlight %}

### Profile

#### step_profile

Complete the *profile.directive.js* directive to load the current user: 

{% highlight js %}
var loggedinUser = AccountService.getLoggedInUser();
profileDialogCtrl.user = {};

angular.copy(loggedinUser,profileDialogCtrl.user);
{% endhighlight %}

We also want to save the profile details, so add the following function too:

{% highlight js %}
profileDialogCtrl.save = function () {

UserService.update(profileDialogCtrl.user).$promise.then(
function(result){
angular.copy(result,loggedinUser);
},
function(error){
	console.log(error);
}
);
$mdDialog.hide();
};
{% endhighlight %}

Extend the template (profile.dialog.html) to load the basic form (between md-dialog-content tags):

{% highlight html %}
<basic-info user="profileDialogCtrl.user"></basic-info>
{% endhighlight %}

In the same template call the function that saves the profile when the user clicks the button:

{% highlight js %}
ng-click="profileDialogCtrl.save()"
{% endhighlight %}

Let's upgrade our sidebar to a whole new level. To do this, inject AccountService in the *leftnav.directive.js* and retrieve the current user:

{% highlight js %}
leftnavCtrl.user = AccountService.getLoggedInUser();
{% endhighlight %}

Let's create a button on the sidenav to reach user profile (*leftnav.html*):

{% highlight html %}
<md-button profile-button class="md-accent" layout="row">
My Profile
</md-button>
{% endhighlight %}

We should also display the loggedin user, so paste the following code between the first md-content tags also in the *leftnav.html* template:

{% highlight html %}
 <avatar class="avatar" user="leftnavCtrl.user"></avatar>
<h3 class="fullname" layout="row" layout-align="center center">{{ leftnavCtrl.user  | fullname }}</h3>
<small class="username" layout="row" layout-align="center center">{{ leftnavCtrl.user.username }}</small>
{% endhighlight %}

Let's put the image of the user in the menu. To do this first inject AccountService in the *menu.directive.js* and query the loggedin user:

{% highlight js %}
menuCtrl.user = AccountService.getLoggedInUser();
{% endhighlight %}

Display the user avatar as a button (menu.html), that way the user can click on it and edit its profile:

{% highlight html %}
    <md-button profile-button  aria-label="More"  ng-if="$root.loggedIn">
      <img ng-src="{{ menuCtrl.user.imageUrl }}" style="     border-radius: 50%;
    margin-top: 10px;   height: 50px;
    background-color: white;"/>
</md-button>
{% endhighlight %}

### Create room

There has been no word about rooms yet, so first let's create the room model as room.model.js file in the room directory:

{% highlight js %}
(function () {
'use strict';

angular
  .module('angularChat')
  .factory('Room', Room);

/** @ngInject */
function Room(apiUrl,$resource) {
  return $resource(apiUrl + '/rooms/:id');
}
})();
{% endhighlight %}

Don't forget about the users of a room, so let's create this model too (room.users.model.js):

{% highlight js %}
(function () {
'use strict';

angular
  .module('angularChat')
  .factory('RoomUser', RoomUser);

/** @ngInject */
function RoomUser(apiUrl,$resource) {
  return $resource(apiUrl + '/rooms/:id/users/:userId', { id: '@_id' });
}
})();
{% endhighlight %}

Next we have to deal with the room service. Create room.service.js file with the following content:

{% highlight js %}
(function () {
'use strict';

angular
  .module('angularChat')
  .service('RoomService', RoomService);

/** @ngInject */
function RoomService($resource, apiUrl,Room, RoomUser) {
    this.createRoom = createRoom;

  function createRoom(room){
    var newRoom = new Room({name : room.name});
    return newRoom.$save();
  }
  }
})();
{% endhighlight %}

The room creator button directive is in the create.room.fab.directive.js, but the event handler method is missing. Let's implement this function:

{% highlight js %}
createRoomCtrl.create = function(){
RoomService.createRoom(createRoomCtrl.newRoom).then(function(result){
  $rootScope.toast("Room "+result._id+" created successfully!");

  $mdDialog.hide();
},function(result){
  console.error(result);
  $rootScope.toast("We couldn't create your room, sorry :(")
});
}
{% endhighlight %}

We need to call this method  in the create.room.fab.tmpl.html template:

{% highlight js %}
ng-click="fabCtrl.createRoomDialog()"
{% endhighlight %}

Add a tooltip too:

{% highlight html %}
<md-tooltip md-direction="left">Create Room</md-tooltip>
{% endhighlight %}

And finally compose the create room dialog template too (create.room.dialog.tmpl.html):

{% highlight html %}
<md-dialog aria-label="New Room">
<form name="createRoomForm">
  <md-toolbar>
    <div class="md-toolbar-tools">
      <h2>Create Room</h2>
      <span flex></span>
      <md-button class="md-icon-button" ng-click="createRoomCtrl.cancel()">
        <md-icon md-svg-src="img/icons/ic_close_24px.svg" aria-label="Close dialog"></md-icon>
      </md-button>
    </div>
  </md-toolbar>
  <md-dialog-content>
     <md-content layout-padding>
      <div layout layout-sm="column">
        <md-input-container flex>
          <label>Room name</label>
          <input ng-enter="createRoomCtrl.create()" ng-model="createRoomCtrl.newRoom.name" required>
        </md-input-container>
      </div>
    </md-content>
  </md-dialog-content>
  <div class="md-actions" layout="column" layout-align="center">
    <md-button class="md-primary md-raised" layout="row" layout-align="center center" ng-click="createRoomCtrl.create()">
      Create
    </md-button>
 </md-dialog-actions>
</form>
</md-dialog>
{% endhighlight %}


### List rooms

In this section we will create a panel that lists all available rooms. This panel will be activated from a button in the top menu. Let's see how to implement all of this!

Add the following features to the RoomService:

* Get all rooms:
{% highlight js %}
this.getRooms = getRooms;
function getRooms() {
    var rooms = Room.query();
    return rooms;
  }
{% endhighlight %}

* Get users of a room given with *roomId* :

{% highlight js %}
this.getUsers = getUsers;

function getUsers(roomId) {
return RoomUser.query({id: roomId});
}

{% endhighlight %}


Now we create a button in the menu (menu/menu.html) that will open the room list on click:

{% highlight html %}
<md-button room-list-opener  ng-if="$root.loggedIn">
Rooms
</md-button>
{% endhighlight %}

This is a plain old Angular Material button with a little bit of directive (*room-list-opener*) on top of it. Wait, we haven't written this directive yet, so do it now (room/room.list.directive.js):

{% highlight js %}
(function() {
'use strict';

angular
  .module('angularChat')
  .directive('roomListOpener', RoomListOpener);

/** @ngInject */
function RoomListOpener($mdBottomSheet) {
  var directive = {
    restrict: 'A',
    link: link
  };
  return directive;

  /** @ngInject */
  function RoomListController(RoomService,$rootScope,$state) {
    var roomListCtrl = this;

  }

  function link(scope, element, attrs) {

   }
}

})();
{% endhighlight %}


We need a click event handler in the directive's link function:

{% highlight js %}
element.on( "click", function($event) {

});
{% endhighlight %}

In the event handler let's use Angular Material' mdBottomSheet component:

{% highlight js %}
$mdBottomSheet.show({
        templateUrl: 'app/room/room.list.tmpl.html',
        controller: RoomListController,
        controllerAs: 'roomListCtrl',
        bindToController: true,
        targetEvent: $event
      });
{% endhighlight %}

Here we can see the previously defined RoomListController listed as controller but it doesn't do anything special yet. We want to display all rooms, so we need to query them from the RoomService

{% highlight js %}
roomListCtrl.rooms = RoomService.getRooms();
{% endhighlight %}

The template is missing though, so add a room.list.tmpl.html file with the following content:

{% highlight html %}
<md-bottom-sheet class="md-list md-has-header">
  <md-subheader>Available Rooms
    <small>Click the room you want to open</small>
  </md-subheader>
  <md-content style="max-height: 200px;">
    <md-chips ng-model="roomListCtrl.rooms" readonly="true">
      <md-chip-template>
        <strong ng-click="roomListCtrl.openRoom($index,$chip)">{{$chip.name}}</strong>
      </md-chip-template>
    </md-chips>
  </md-content>
</md-bottom-sheet>
{% endhighlight %}

### Open room

In this section we will create a tab layout for the opened rooms.

First things first, add a *getRoom* function to the *room.service.js* file:

{% highlight js %}
this.getRoom = getRoom;

function getRoom(roomId) {
var room = Room.get({id: roomId});
return room;
}
{% endhighlight %}

We also need a service that can keep tracks of opened rooms (*opened.room.factory.js*):

{% highlight js %}
(function () {
'use strict';

angular
  .module('angularChat')
  .factory('openedRoomsFactory', openedRoomsFactory);

/** @ngInject */
function openedRoomsFactory(RoomService,$localStorage,$rootScope) {
  var self = this;

  var roomFactoryObj = {};

  return roomFactoryObj;
}

})();
{% endhighlight %}

Add the following functions to the opened.rooms.factory.js:

* Index of currently selected room:

{% highlight js %}
  roomFactoryObj.setSelectedIndex = function(newIndex){
    $localStorage.index  = newIndex;
  };

  roomFactoryObj.getSelectedIndex = function(){
    return $localStorage.index;
  };
{% endhighlight %}

* Get room by index:
{% highlight js %}
  roomFactoryObj.getRoomByIndex = function(index){
    var room = _.findWhere($localStorage.rooms, { 'index': index });
    return room;
  };
{% endhighlight %}

* Get all opened room:
{% highlight js %}
 roomFactoryObj.getRooms = function(){
 return $localStorage.rooms;
 };
{% endhighlight %}

* Add room to opened rooms:
{% highlight js %}
  roomFactoryObj.addRoom = function(room){
     if(typeof $localStorage.rooms == "undefined") {
            $localStorage.rooms = [];
          }
    
          room.index = $localStorage.rooms.length;
          $localStorage.rooms.push(room);
          $localStorage.index = room.index;
          $rootScope.$emit("room.added");
  };
{% endhighlight %}

* Remove room from the opened rooms:
{% highlight js %}
roomFactoryObj.removeRoom = function(index){
    $localStorage.rooms.splice(index, 1);
         _.forEach($localStorage.rooms, function(room, i) {
           room.index = i;
         });
  };
{% endhighlight %}

* Is there any opened room?
{% highlight js %}
roomFactoryObj.hasRoom = function(){
    return $localStorage.rooms && $localStorage.rooms.length>0;
  };
{% endhighlight %}

* Is the room opened?
{% highlight js %}
 roomFactoryObj.containsRoom = function(room){
    var result = _.findWhere($localStorage.rooms, { '_id': room._id });
    return typeof result != "undefined";
  };
{% endhighlight %}

* Synchronize previously opened rooms from local storage when the factory is created:
{% highlight js %}
function syncRoomsFromLocalStorage(){
    _.forEach($localStorage.rooms,function(storedRoom){
      $localStorage.rooms[storedRoom._id] = RoomService.getRoom(storedRoom._id)
    });
  }

  syncRoomsFromLocalStorage();
{% endhighlight %}

When you click a room in the list, it would be awesome to actually open the selected room. That means a new navigation aka state in the index.route.js file:

{% highlight js %}
.state('rooms.room', {
        url: '/:id',
        templateUrl: 'app/room/room.item.html',
        controller: 'RoomItemController',
        controllerAs: 'roomItemCtrl',
        data: {authenticated: true},
        resolve: {
          users : function (allUsersFactory) {
            return allUsersFactory.initUsers();
          },
          room: function (RoomService, $stateParams) {
            var roomId = $stateParams.id;
            return RoomService.getRoom(roomId).$promise;
          },
          messages: function (MessageService, $state, $stateParams) {
            var roomId = $stateParams.id;
            return MessageService.getRoomMessages(roomId).$promise;
          }
        }
      })
{% endhighlight %}

As you can see we also need a RoomItemController for this state, so create room.item.controller.js file in the room directory with the following content:

{% highlight js %}
(function () {
'use strict';

angular
  .module('angularChat')
  .controller("RoomItemController", RoomItemController);

/** @ngInject */
function RoomItemController($scope, users, room, messages, MessageService, RoomService, $log, $rootScope, $state, openedRoomsFactory, apiUrl, socketFactory, AccountService, Message, allUsersFactory) {
    var roomItemCtrl = this;
     roomItemCtrl.newMessage = "";
     var roomId = $state.params.id;
     roomItemCtrl.messages = messages;
     roomItemCtrl.users = [];
     roomItemCtrl.allusers = allUsersFactory.users;

   _.forEach(room.users, function (userId) {
        if (!isUserAlreadyAdded(userId)) {
          roomItemCtrl.users.push(allUsersFactory.users[userId]);
        }
      });
  
      roomItemCtrl.room = room;

  function isUserAlreadyAdded(userId){
    var userfound = _.findWhere(roomItemCtrl.users, {'_id': userId});
    return typeof userfound != "undefined";
  }
}
})();
{% endhighlight %}

Don't forget to subscribe on your favourite socket.io events in the controller:

{% highlight js %}
 socketFactory.emit("subscribe", {room: roomId, user: AccountService.getLoggedInUser()});

    socketFactory.on("user.joined", function (user) {
      if (!isUserAlreadyAdded(user._id)) {
        roomItemCtrl.users.push(user);
      }
    });

    socketFactory.on("user.left", function (user) {
      _.remove(roomItemCtrl.users, {
        _id: user._id
      });
    });

    $scope.$on("$destroy", function () {
      socketFactory.emit("unsubscribe", {room: roomId, user: AccountService.getLoggedInUser()});
    });
{% endhighlight %}

Next create the basic room view in room.item.html file:

{% highlight html %}
<div layout="row" layout-wrap>
<md-content layout="column" flex="80" flex-sm="100">
  <md-content style="height: 60vh;">

</md-content>
  <md-content layout-padding layout="column">

  </md-content>
</md-content>
<md-content class="side-nav room-users" hide-sm layout="column" flex="18">
  <md-list layout-fill>
    <md-subheader class="md-accent">Available users</md-subheader>
   <md-list-item class="md-2-line contact-item selected" ng-repeat="(index, contact) in roomItemCtrl.users track by contact._id" ng-show="contact.username">
           <avatar user="contact"></avatar>
           <div class="md-list-item-text compact">
             <h3>{{contact | fullname }}</h3>
             <p>@{{contact.username}}</p>
           </div>
           <md-divider></md-divider>
         </md-list-item>
  </md-list>
</md-content>
</div>
{% endhighlight %}

Declare the *openedRoomsFactory* dependency in room.list.directive.js file, and implement the function responsible for opening a room:

{% highlight js %}
  roomListCtrl.openRoom = function(index,room){
if(!openedRoomsFactory.containsRoom(room)){
  openedRoomsFactory.addRoom(room);
  $state.go("rooms.room",{id: room._id},{reload: false});
}else{
  $rootScope.toast("You've already opened this room!");
}
};
{% endhighlight %}

Add an event handler to room.list.tmpl.html:

{% highlight js %}
ng-click="roomListCtrl.openRoom($index,$chip)"
{% endhighlight %}

For our tab layout we need to create the RoomsController too (room.tabs.controller.js):

room.tabs.controller.js:

{% highlight js %}
(function () {
'use strict';

angular
  .module('angularChat')
  .controller('RoomsController', RoomsController);

/** @ngInject */
function RoomsController($scope, $timeout, $mdBottomSheet, toastr, RoomService, $log,$rootScope,$state,openedRoomsFactory) {
  var roomsCtrl = this;

   }
})();
{% endhighlight %}

In the controller we need to synchronize the opened rooms:

{% highlight js %}
syncFromOpenedRoomsFactory();

$rootScope.$on("room.added", onRoomAdded);

    function onRoomAdded() {
      syncFromOpenedRoomsFactory();
      goToRoom(roomsCtrl.selectedIndex);
    }

    function syncFromOpenedRoomsFactory(){
      roomsCtrl.selectedIndex = openedRoomsFactory.getSelectedIndex();
      roomsCtrl.rooms = openedRoomsFactory.getRooms();
    }

    function goToRoom(newIndex) {
      if(openedRoomsFactory.hasRoom()){
        $state.go("rooms.room",{id: openedRoomsFactory.getRoomByIndex(newIndex)._id});
      }else{
        $state.go("rooms");
      }
    }
{% endhighlight %}

When you close a tab, you have to remove the room from the factory:

{% highlight js %}
roomsCtrl.removeRoom = function (index) {
    openedRoomsFactory.removeRoom(index);
      goToRoom(openedRoomsFactory.getRooms().length-1);
  };
{% endhighlight %}

The current index and navigation state has to be updated too when you click on a new tab:

{% highlight js %}
 $scope.$watch("roomsCtrl.selectedIndex",function(newIndex){
      openedRoomsFactory.setSelectedIndex(newIndex);
      goToRoom(newIndex);
    });
{% endhighlight %}

Next create the view for the tab layout (room.tabs.tmpl.html) with the following code:

{% highlight html %}
<md-content flex ng-if="roomsCtrl.rooms.length > 0">
<md-subheader>Opened Rooms right now</md-subheader>
  <md-tabs md-dynamic-height md-selected="roomsCtrl.selectedIndex" md-border-bottom>
  <md-tab ng-repeat="room in roomsCtrl.rooms">
    <md-tab-label>{{room.name}} <a ng-click="roomsCtrl.removeRoom($index,room)">
      <md-icon md-svg-icon="navigation:close"></md-icon>
    </a>
    </md-tab-label>
     <md-tab-body>
            <div ng-if="roomsCtrl.selectedIndex === $index" ui-view flex></div>
          </md-tab-body>
</md-tabs>
</md-content>
{% endhighlight %}

Don't forget about unsubscribing the user when leaving the room! To achieve this, add the following changes to the logout method of the account.service.js file (add socketFactory and openedRoomsFactory dependencies too):

{% highlight js %}
var openedRooms = openedRoomsFactory.getRooms();
//logout from rooms
_.forEach(openedRooms, function (room) {
socketFactory.emit("unsubscribe", {room: room._id, user: getLoggedInUser()});
});
{% endhighlight %}

Remove opened rooms too:

{% highlight js %}
delete $localStorage.rooms;
delete $localStorage.index;
{% endhighlight %}

Routing needs to be extended in the index.routes.js file. There is a state called "rooms" already, add to this the following changes:

{% highlight js %}
.state('rooms', {
  url: '/rooms',
  templateUrl: 'app/room/room.tabs.tmpl.html',
  controller: 'RoomsController',
  controllerAs: 'roomsCtrl',
  data: {authenticated: true}
});
{% endhighlight %}

Add openedRoomsFactory dependency to create.room.fab.directive.js and following changes too (here you make sure when opening a new room it will be listed in the opened rooms and actually opened too):

{% highlight js %}
openedRoomsFactory.addRoom(result);
$state.go("rooms.room",{id: result._id});
{% endhighlight %}


### Messages

First create a directory called "message" for the message aspect. Then add a message.model.js file in this directory with the following content:

{% highlight js %}
(function () {
'use strict';

angular.module('angularChat')
.factory('Message', Message);

/** @ngInject */
function Message(apiUrl,$resource) {
  return $resource(apiUrl + '/rooms/:id/messages/:messageId',
{ id: '@_id' });
}

})();
{% endhighlight %}

Next create the service that is responsible for message handling (message.service.js):

{% highlight js %}
(function() {
'use strict';

angular
  .module('angularChat')
  .service('MessageService',MessageService);

/** @ngInject */
function MessageService($resource,apiUrl,Message,Room,AccountService) {
}
})();
{% endhighlight %}

Add the following functions to the service:

* Get Messages in a room:
{% highlight js %}
function getRoomMessages(roomId){
	return Message.query({id: roomId});
}
{% endhighlight %}

* Create message in a room:
{% highlight js %}
function createRoomMessage(roomId, message){
var newMessage = new Message();
newMessage.text = message;
newMessage.user = AccountService.getLoggedInUser();
newMessage.$save({id: roomId});
}
{% endhighlight %}

To make these two functions available from the outside add the following statements:

{% highlight js %}
this.getRoomMessages = getRoomMessages;
this.createRoomMessage = createRoomMessage;
{% endhighlight %}

Add a function that handles message creation in room.item.controller.js:

{% highlight js %}
roomItemCtrl.createMessage = function () {
MessageService.createRoomMessage(roomId, roomItemCtrl.newMessage);
roomItemCtrl.newMessage = "";
};
{% endhighlight %}

Add the missing socket handling:

{% highlight js %}
socketFactory.on("new message",function (message) {
roomItemCtrl.messages.push(message);
});
{% endhighlight %}

Open room/room.item.html template! Here create an Angular Material list, that will display the roomItemCtrl.messages variable:

{% highlight html %}
 <md-list scroll="roomItemCtrl.messages">
        <md-subheader class="md-info">Messages in room {{ roomItemCtrl.room.name }}</md-subheader>
        <message ng-repeat="message in roomItemCtrl.messages track by message._id" message="message"
                 author="roomItemCtrl.allusers[message.authorId]"
                 ng-class="{ 'repeated-author' : $index>0 && message.authorId == roomItemCtrl.messages[$index-1].authorId}"></message>
      </md-list>
{% endhighlight %}


Create new message:

{% highlight html %}
<md-input-container class="md-accent">
<label>New Message</label>
<input ng-enter="roomItemCtrl.createMessage()" ng-model="roomItemCtrl.newMessage" md-maxlength="350"/>
</md-input-container>
{% endhighlight %}

Previously we used message directive, but it doesn't really exist. So create message.directive.js in the message folder:

{% highlight js %}
(function() {
'use strict';

angular
  .module('angularChat')
  .directive('message', message);

/** @ngInject */
function message() {
  var directive = {
    restrict: 'E',
    templateUrl: 'app/message/message.item.tmpl.html',
    controller: MessageController,
    controllerAs: 'messageCtrl',
    bindToController: true,
    scope:{
      message: "=",
      author: "="
    }
  };

  return directive;

  /** @ngInject */
  function MessageController($mdDialog){
    var messageCtrl = this;
  }
}

})();
{% endhighlight %}

As last step create the template for that directive with the following content:

{% highlight html %}
<md-list-item class="contact-item md-2-line selected" ng-if="messageCtrl.author">

  <avatar user="messageCtrl.author"  class="md-avatar message-avatar"></avatar>

  <div class="md-list-item-text compact">
    <p style="text-align: right;"><small><strong>{{messageCtrl.author.username | username }}</strong>, {{ messageCtrl.message | createdDate }}</small></p>
     <div class="message-content">
          <ng-embed
            embed-data="messageCtrl.message.text" embed-options="$root.embedOptions"
          ></ng-embed>
        </div>
  </div>
</md-list-item>

{% endhighlight %}

Now you are ready, and hopefully you can use your brand new chat application.

If you stuck at some step, you can checkout the proper branch to this step from the Git repository.

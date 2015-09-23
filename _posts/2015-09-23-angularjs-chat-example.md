---
layout: post
title: AngularJS Chat
---

##Feladat
A gyakorlat során a cél egy chat alkalmazás létrehozása AngularJS, Angular Material, Socket IO felhasználásával. 
 
###A fő funkciók: 
* Login
* Regisztráció
* Logout	
* Profil megtekintése, módosítása
* Új szoba létrehozása
* Szobák listázása
* Szoba megnyitása
* szobában lévő felhasználók megjelenítése
* üzenetek megjelenítése
* új üzenet írása

## Forráskód

### Forráskód elérése Git-el
Git esetén a következő paranccsal klónozzuk a repót: 
<pre><code>git clone https://github.com/annatomka/angular-chat.git</code></pre>

###Letöltés zip-ként
Ha nincs Git, akkor a következő linkről zip-ként is letölthetjük:

https://github.com/annatomka/angular-chat/archive/step_registration.zip
Ezt tömörítsük ki egy tetszőleges mappába. 

Importáljuk be a projektet Intellij-be.

### Regisztráció

A kiindulási állapotban van egy login formunk, de sajnos nincs userünk mellyel be tudnánk lépni a chatbe. Ebben a szakaszban a regisztrációs formot készítjük el. 
Hozzuk létre az index.routes.js-ben a route-ot: 

<pre><code>
.state('registration', {
url: '/registration',
templateUrl: 'app/registration/registration.html',
controller: 'RegistrationController',
controllerAs: 'registrationCtrl'
})
</code></pre>

A kódból látható, hogy kell készítenünk egy template-et és egy controllert is…
Hozzuk létre a registration mappát. Ebben pedig a registration.controller.js fájlt a következő tartalommal: 

<pre><code>
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
</code></pre>

A registration.html fájl tartalma: 

<pre>
<code>
&#x3C;div class=&#x22;registration-frame&#x22; ui-view=&#x22;&#x22; flex=&#x22;&#x22; layout=&#x22;row&#x22;&#x3E;
&#x3C;div layout=&#x22;row&#x22; flex=&#x22;&#x22; layout-padding=&#x22;&#x22; layout-fill=&#x22;&#x22; layout-align=&#x22;center center&#x22; class=&#x22;ng-scope&#x22;&#x3E;
  &#x3C;div flex=&#x22;40&#x22; flex-lg=&#x22;50&#x22; flex-md=&#x22;70&#x22; flex-sm=&#x22;100&#x22;&#x3E;
    &#x3C;md-card class=&#x22;md-cyan-theme&#x22;&#x3E;
      &#x3C;md-toolbar class=&#x22;md-cyan-theme&#x22; layout-padding&#x3E;
        &#x3C;div layout=&#x22;column&#x22; layout-align=&#x22;center&#x22; class=&#x22;padding-20 text-center&#x22;&#x3E;
          &#x3C;img ng-src=&#x22;img/logo.png&#x22;&#x3E;
          &#x3C;h1 layout=&#x22;row&#x22; layout-align=&#x22;center center&#x22; class=&#x22;md-headline ng-scope&#x22; translate=&#x22;&#x22;&#x3E;Registration&#x3C;/h1&#x3E;
        &#x3C;/div&#x3E;
      &#x3C;/md-toolbar&#x3E;
      &#x3C;md-content class=&#x22;md-padding md-cyan-theme&#x22;&#x3E;
        &#x3C;form name=&#x22;registration&#x22;&#x3E;
          &#x3C;basic-info user=&#x22;registrationCtrl.user&#x22;&#x3E;&#x3C;/basic-info&#x3E;
          &#x3C;div class=&#x22;md-actions&#x22; layout=&#x22;column&#x22; layout-align=&#x22;center&#x22;&#x3E;
            &#x3C;md-button class=&#x22;md-raised md-primary&#x22; layout=&#x22;row&#x22; layout-align=&#x22;center center&#x22; ng-disabled=&#x22;registration.$invalid&#x22; ng-click=&#x22;registrationCtrl.register()&#x22;&#x3E;
              Registration
            &#x3C;/md-button&#x3E;
            &#x3C;a
              class=&#x22;md-primary md-button md-cyan-theme&#x22; ui-sref=&#x22;login&#x22;
              aria-label=&#x22;Already registered? Login now&#x22;&#x3E;Already registered? Login now&#x3C;/a&#x3E;
          &#x3C;/div&#x3E;

        &#x3C;/form&#x3E;
      &#x3C;/md-content&#x3E;
    &#x3C;/md-card&#x3E;
  &#x3C;/div&#x3E;
&#x3C;/div&#x3E;
&#x3C;/div&#x3E;
</code>
</pre>

A login.html-ben vegyünk fel egy regisztrációs gombot: 
<pre><code>
&#x3C;a
class=&#x22;md-primary md-button md-cyan-theme&#x22; ui-sref=&#x22;registration&#x22;
aria-label=&#x22;Don&#x27;t have an account? Create one now&#x22;&#x3E;Don&#x27;t have an account? Create one now&#x3C;/a&#x3E;
</code></pre> 
 
### Profil

#### step_profile

Ha nem sikerült volna az előző szakaszt végigvinni, a step_profile branch-re válts át és innen folytasd a munkát:
<pre><code>
git checkout step_profile
</code></pre>

Egészítsük ki a direktívát (profile.directive.js) hogy betöltse az aktuális usert:  

<pre><code>
var loggedinUser = AccountService.getLoggedInUser();
profileDialogCtrl.user = {};

angular.copy(loggedinUser,profileDialogCtrl.user);
</code></pre>

Elmentési lehetőség: 

<pre><code>
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
</code></pre>

Egészítsük ki a template-et (profile.dialog.html) hogy betöltse a basic formot (az md-dialog-content tagek közé):

<pre><code>
&#x3C;basic-info user=&#x22;profileDialogCtrl.user&#x22;&#x3E;&#x3C;/basic-info&#x3E;
</code></pre>

Ugyanitt hívjuk meg a profilt mentő függvényt a gomb click eseményekor: 

<pre><code>
ng-click="profileDialogCtrl.save()"
</code></pre>

Egy kis oldalsáv tuning következik. 
A leftnav.directive.js-ben injektáljuk az AccountService-t és kérjük le az aktuális usert: 

<pre><code>
leftnavCtrl.user = AccountService.getLoggedInUser();
</code></pre>

hozzunk létre egy gombot a sidenavon (leftnav.html): 

<pre><code>
&#x3C;md-button profile-button class=&#x22;md-accent&#x22; layout=&#x22;row&#x22;&#x3E;
My Profile
&#x3C;/md-button&#x3E;
</code></pre>

Ha már úgyis itt vagyunk, jelenítsük meg a bejelentkezett usert (az első md-content közé): 

<pre><code>
&#x3C;img class=&#x22;avatar&#x22; ng-src=&#x22;{{ leftnavCtrl.user.imageUrl }}&#x22;/&#x3E;
&#x3C;h3 class=&#x22;fullname&#x22; layout=&#x22;row&#x22; layout-align=&#x22;center center&#x22;&#x3E;{{ leftnavCtrl.user  | fullname }}&#x3C;/h3&#x3E;
&#x3C;small class=&#x22;username&#x22; layout=&#x22;row&#x22; layout-align=&#x22;center center&#x22;&#x3E;@{{ leftnavCtrl.user.username}}&#x3C;/small&#x3E;
</code></pre>

A menübe is tegyük ki a user képét, amire kattintva bejön a profiloldal. Ehhez a menu.directive.js-ben AccountService injektálása és:

<pre><code>
menuCtrl.user = AccountService.getLoggedInUser();
</code></pre>

menu.html-ben: 
<pre><code>
&#x3C;md-button profile-button class=&#x22;md-icon-button&#x22; aria-label=&#x22;More&#x22;  ng-if=&#x22;$root.loggedIn&#x22;&#x3E;
&#x3C;img ng-src=&#x22;{{ menuCtrl.user.imageUrl }}&#x22; class=&#x22;md-avatar message-avatar&#x22; style=&#x22;    height: 50px;
background-color: white;&#x22;/&#x3E;
&#x3C;/md-button&#x3E;
</code></pre>

### Szoba létrehozása 

#### step_create_room

Szobákról még nem volt szó, elsőként a modellt alkossuk meg: room.model.js a room mappában.

<pre><code>
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
</code></pre>

A room-ban lehetnek user-ek is, így ennek a modelljét se hagyjuk ki! room.users.model.js: 

<pre><code>
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
</code></pre>

Következik a room service. Hozzuk létre a room.service.js fájlt a következő tartalommal: 

<pre><code>
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
</code></pre>

A szobát létrehozó gomb direktívája a create.room.fab.directive.js-ben található. Innen hiányzik a click eseményt kezelő függvény. Egészítsük ki vele a direktívát:

<pre><code>
createRoomCtrl.create = function(){
RoomService.createRoom(createRoomCtrl.newRoom).then(function(result){
  $rootScope.toast("Room "+result._id+" created successfully!");

  $mdDialog.hide();
},function(result){
  console.error(result);
  $rootScope.toast("We couldn't create your room, sorry :(")
});
}
</code></pre>

template: create.room.fab.tmpl.html egészítsük ki az eseménykezelővel: 

<pre><code>
ng-click="fabCtrl.createRoomDialog()"
</code></pre>

adjunk egy tooltipet is neki:

<pre><code>
&#x3C;md-tooltip md-direction=&#x22;left&#x22;&#x3E;Create Room&#x3C;/md-tooltip&#x3E;
</code></pre>

create.room.dialog.tmpl.html-ben:

<pre><code>
&#x3C;md-dialog aria-label=&#x22;New Room&#x22;&#x3E;
&#x3C;form name=&#x22;createRoomForm&#x22;&#x3E;
  &#x3C;md-toolbar&#x3E;
    &#x3C;div class=&#x22;md-toolbar-tools&#x22;&#x3E;
      &#x3C;h2&#x3E;Create Room&#x3C;/h2&#x3E;
      &#x3C;span flex&#x3E;&#x3C;/span&#x3E;
      &#x3C;md-button class=&#x22;md-icon-button&#x22; ng-click=&#x22;createRoomCtrl.cancel()&#x22;&#x3E;
        &#x3C;md-icon md-svg-src=&#x22;img/icons/ic_close_24px.svg&#x22; aria-label=&#x22;Close dialog&#x22;&#x3E;&#x3C;/md-icon&#x3E;
      &#x3C;/md-button&#x3E;
    &#x3C;/div&#x3E;
  &#x3C;/md-toolbar&#x3E;
  &#x3C;md-dialog-content&#x3E;
     &#x3C;md-content layout-padding&#x3E;
      &#x3C;div layout layout-sm=&#x22;column&#x22;&#x3E;
        &#x3C;md-input-container flex&#x3E;
          &#x3C;label&#x3E;Room name&#x3C;/label&#x3E;
          &#x3C;input ng-enter=&#x22;createRoomCtrl.create()&#x22; ng-model=&#x22;createRoomCtrl.newRoom.name&#x22; required&#x3E;
        &#x3C;/md-input-container&#x3E;
      &#x3C;/div&#x3E;
    &#x3C;/md-content&#x3E;
  &#x3C;/md-dialog-content&#x3E;
  &#x3C;div class=&#x22;md-actions&#x22; layout=&#x22;column&#x22; layout-align=&#x22;center&#x22;&#x3E;
    &#x3C;md-button class=&#x22;md-primary md-raised&#x22; layout=&#x22;row&#x22; layout-align=&#x22;center center&#x22; ng-click=&#x22;createRoomCtrl.create()&#x22;&#x3E;
      Create
    &#x3C;/md-button&#x3E;
  &#x3C;/div&#x3E;
&#x3C;/form&#x3E;
&#x3C;/md-dialog&#x3E;
</code></pre>

 
### Szobák listázása

#### step_list_rooms

Ebben a szakaszban létrehozunk egy szobákat listázó panelt, amely a menüben lévő gombra kattintva kúszik fel az oldal alján. Lássuk!

Egészítsük ki a RoomService-t:
* az összes szoba lekérdezésére: 

<pre><code>
this.getRooms = getRooms;
function getRooms() {
    var rooms = Room.query();
    return rooms;
  }
</code></pre>

* szoba felhasználóinak lekérdezése:

<pre><code>
this.getUsers = getUsers;

function getUsers(roomId) {
return RoomUser.query({id: roomId});
}

</code></pre>

Hozzunk létre a menüben egy gombot, amire felnyílik majd a szobalista: 
menu/menu.html

<pre><code>
&#x3C;md-button room-list-opener  ng-if=&#x22;$root.loggedIn&#x22;&#x3E;
Rooms
&#x3C;/md-button&#x3E;
</code></pre>

Ez egy sima angularmaterial-os gomb, kiegészítve egy direktívával, amit még nem írtunk meg, így tegyük meg azt is.


room/room.list.directive.js létrehozása:

<pre><code>
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
</code></pre>



A link függvénybe tegyünk egy click eseménykezelőt:  

<pre><code>
element.on( "click", function($event) {
	 
});
</code></pre>

Az eseménykezelőben pedig használjuk az mdBottomSheet komponensét az angular materialnak:

<pre><code>
$mdBottomSheet.show({
        templateUrl: 'app/room/room.list.tmpl.html',
        controller: RoomListController,
        controllerAs: 'roomListCtrl',
        bindToController: true,
        targetEvent: $event
      });
</code></pre>

Itt használjuk a már definiált RoomListControllert, de nem csinál még semmit sem. Meg szeretnénk jeleníteni az összes szobát, kérjük hát le a RoomService-től:

<pre><code>
roomListCtrl.rooms = RoomService.getRooms();
</code></pre>

Már csak a room.list.tmpl.html hiányzik, hozzuk létre a következő tartalommal:

<pre><code>
&#x3C;md-bottom-sheet class=&#x22;md-list md-has-header&#x22;&#x3E;
&#x3C;md-subheader&#x3E;Available Rooms &#x3C;small&#x3E;Click the room you want to open&#x3C;/small&#x3E;&#x3C;/md-subheader&#x3E;
&#x3C;md-chips ng-model=&#x22;roomListCtrl.rooms&#x22; readonly=&#x22;true&#x22;&#x3E;
  &#x3C;md-chip-template&#x3E;
    &#x3C;strong&#x3E;{{$chip.name}}&#x3C;/strong&#x3E;
  &#x3C;/md-chip-template&#x3E;
&#x3C;/md-chips&#x3E;
&#x3C;/md-bottom-sheet&#x3E;
</code></pre>

### Szoba megnyitása

#### step_room_view

Ebben a szakaszban a cél, hogy egy tabos elrendezést kialakítsunk a megnyitott szobáknak. Először is a room.service.js fájlt egészítsük ki egy szobát lekérő metódussal: 

<pre><code>
this.getRoom = getRoom;

function getRoom(roomId) {
var room = Room.get({id: roomId});
return room;
}
</code></pre>

Létre kell hoznunk egy olyan service-t ami nyílvántartja a már megnyitott szobákat: opened.room.factory.js

<pre><code>
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
</code></pre>

Egészítsük ki a következő függvényekkel: 
* Aktuálisan megnyitott szoba index:

<pre><code>
  roomFactoryObj.setSelectedIndex = function(newIndex){
    $localStorage.index  = newIndex;
  };

  roomFactoryObj.getSelectedIndex = function(){
    return $localStorage.index;
  };
</code></pre>

* Szoba lekérése index alapján: 
<pre><code>
  roomFactoryObj.getRoomByIndex = function(index){
    var room = _.findWhere($localStorage.rooms, { 'index': index });
    return room;
  };
</code></pre>

* Összes megnyitott szoba lekérdezése
<pre><code>
 roomFactoryObj.getRooms = function(){
 return $localStorage.rooms;
 };
</code></pre>

* Szoba hozzáadása a már megnyitott szobákhoz:
<pre><code>
  roomFactoryObj.addRoom = function(room){
    if(typeof $localStorage.rooms == "undefined") {
      $localStorage.rooms = [];
    }

    room.index = $localStorage.rooms.length;
    $localStorage.index = room.index;
    $localStorage.rooms.push(room);
    $rootScope.$emit("room.added");
  };
</code></pre>

* Szoba eltávolítása a már megnyitott szobák közül:
<pre><code>
roomFactoryObj.removeRoom = function(index){
    $localStorage.rooms.splice(index, 1);
  };
</code></pre>

* Van-e megnyitott szoba?
<pre><code>
roomFactoryObj.hasRoom = function(){
    return $localStorage.rooms && $localStorage.rooms.length>0;
  };
</code></pre>

* Meg van-e nyitva egy szoba?
<pre><code>
 roomFactoryObj.containsRoom = function(room){
    var result = _.findWhere($localStorage.rooms, { '_id': room._id });
    return typeof result != "undefined";
  };
</code></pre>

* A factory létrejöttekor szinkronizáljunk a korábban megnyitott szobákat a local storage-ból: 
<pre><code>
function syncRoomsFromLocalStorage(){
    _.forEach($localStorage.rooms,function(storedRoom){
      $localStorage.rooms[storedRoom._id] = RoomService.getRoom(storedRoom._id)
    });
  }

  syncRoomsFromLocalStorage();
</code></pre>

A szoba listán ha rákattintunk egy elemre akkor szeretnénk megnyitni egy szobát. Ezzel egy új navigációhoz jutunk, így az index.route.js-ben: 
<pre><code>
.state('rooms.room', {
url: '/:id',
templateUrl: 'app/room/room.item.html',
controller: 'RoomItemController',
controllerAs: 'roomItemCtrl',
data: {authenticated: true}
})
</code></pre>

hozzuk létre az itt említett RoomItemControllert: room.item.controller.js fájl létrehozása a room mappában a következő tartalommal: 

<pre><code>
(function () {
'use strict';

angular
  .module('angularChat')
  .controller("RoomItemController", RoomItemController);

/** @ngInject */
function RoomItemController($scope, $timeout, $mdBottomSheet, toastr, RoomService, $log, $rootScope, $state, openedRoomsFactory, apiUrl, socketFactory, AccountService, allUsersFactory) {
  var roomItemCtrl = this;
  var roomId = $state.params.id;

  roomItemCtrl.users = [];
  roomItemCtrl.allusers = allUsersFactory.users;

  RoomService.getRoom(roomId).$promise.then(function (room) {
    _.forEach(room.users, function (userId) {
      if (!isUserAlreadyAdded(userId)) {
        roomItemCtrl.users.push(allUsersFactory.users[userId]);
      }
    });

    roomItemCtrl.room = room;
  });

  function isUserAlreadyAdded(userId){
    var userfound = _.findWhere(roomItemCtrl.users, {'_id': userId});
    return typeof userfound != "undefined";
  }
}
})();
</code></pre>

Ne feledkezzünk meg a kedvenc socket.io-s eseményekre feliratkozni a kontrollerben! :)

<pre><code>
socketFactory.emit("subscribe", {room: roomId, user: AccountService.getLoggedInUser()});

  socketFactory.on("user.joined",function (user) {
      if (!isUserAlreadyAdded(user._id)) {
        roomItemCtrl.users.push(user);
      }
  });

  socketFactory.on("user.left",function (user) {
      _.remove(roomItemCtrl.users, {
        _id: user._id
      });
  });

  $scope.$on("$destroy", function () {
    socketFactory.emit("unsubscribe", {room: roomId, user: AccountService.getLoggedInUser()});
  });
</code></pre>

Készítsük el a szoba megjelenítésének alapjait. room.item.html létrehozása:

<pre><code>
&#x3C;div layout=&#x22;row&#x22; layout-wrap&#x3E;
&#x3C;md-content layout=&#x22;column&#x22; flex=&#x22;80&#x22; flex-sm=&#x22;100&#x22;&#x3E;
  &#x3C;md-content style=&#x22;height: 60vh;&#x22;&#x3E;
  
&#x3C;/md-content&#x3E;
  &#x3C;md-content layout-padding&#x3E;

  &#x3C;/md-content&#x3E;
&#x3C;/md-content&#x3E;
&#x3C;md-content class=&#x22;side-nav room-users&#x22; hide-sm layout=&#x22;column&#x22; flex=&#x22;18&#x22;&#x3E;
  &#x3C;md-list layout-fill&#x3E;
    &#x3C;md-subheader class=&#x22;md-accent&#x22;&#x3E;Available users&#x3C;/md-subheader&#x3E;
    &#x3C;md-list-item class=&#x22;md-2-line contact-item selected&#x22; ng-repeat=&#x22;(index, contact) in roomItemCtrl.users&#x22;&#x3E;
      &#x3C;img ng-src=&#x27;{{ contact.imageUrl }}&#x27; class=&#x22;md-avatar&#x22; alt=&#x22;{{contact.name}}&#x22;/&#x3E;
      &#x3C;div class=&#x22;md-list-item-text compact&#x22;&#x3E;
        &#x3C;h3&#x3E;{{contact | fullname }}&#x3C;/h3&#x3E;
        &#x3C;p&#x3E;@{{contact.username}}&#x3C;/p&#x3E;
      &#x3C;/div&#x3E;
      &#x3C;md-divider&#x3E;&#x3C;/md-divider&#x3E;
    &#x3C;/md-list-item&#x3E;
  &#x3C;/md-list&#x3E;
&#x3C;/md-content&#x3E;
&#x3C;/div&#x3E;
</code></pre>

room.list.directive.js fájlban vegyük fel az openedRoomsFactory függőséget és definiáljuk a szoba megnyitásáárét felelős függvényt:

<pre><code>
  roomListCtrl.openRoom = function(index,room){
if(!openedRoomsFactory.containsRoom(room)){
  openedRoomsFactory.addRoom(room);
  $state.go("rooms.room",{id: room._id},{reload: false});
}else{
  $rootScope.toast("You've already opened this room!");
}
};
</code></pre>

room.list.tmpl.html fájlban eseménykezelő hozzáadása: 

<pre><code>
ng-click="roomListCtrl.openRoom($index,$chip)"
</code></pre>

A tabos megjelenítéshez hozzunk létre egy RoomsController-t! 

room.tabs.controller.js: 

<pre><code>
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
</code></pre>

Olvassuk be ha már volt elmentett szoba:

<pre><code>
syncFromOpenedRoomsFactory();

function syncFromOpenedRoomsFactory(){
    roomsCtrl.selectedIndex = openedRoomsFactory.getSelectedIndex();
    roomsCtrl.rooms = openedRoomsFactory.getRooms();
  }
</code></pre>

Ha új szobát nyitunk meg, szinkronizáljuk a mezőket:

<pre><code>
$rootScope.$on("room.added",syncFromOpenedRoomsFactory);
</code></pre>

Tab bezárásakor szoba eltávolítása:

<pre><code>
roomsCtrl.removeRoom = function (index) {
    openedRoomsFactory.removeRoom(index);
  };
</code></pre>

Tabokat ha kattintgatjuk, akkor frissítsük az indexet az openedRoomsFactory-ban is!
A routingot is állítsuk be ennek megfelelően!

<pre><code>
$scope.$watch("roomsCtrl.selectedIndex",function(newIndex){
    openedRoomsFactory.setSelectedIndex(newIndex);
    if(openedRoomsFactory.hasRoom()){
      $state.go("rooms.room",{id: openedRoomsFactory.getRoomByIndex(newIndex)._id});
    }
  });
</code></pre>

Hozzuk létre a tabos elrendezés nézetét: room.tabs.tmpl.html a következő tartalommal:

<pre><code>
&#x3C;md-content flex ng-if=&#x22;roomsCtrl.rooms.length &#x3E; 0&#x22;&#x3E;
&#x3C;md-subheader&#x3E;Opened Rooms right now&#x3C;/md-subheader&#x3E;
&#x3C;md-tabs md-dynamic-height md-selected=&#x22;roomsCtrl.selectedIndex&#x22; md-border-bottom md-autoselect&#x3E;
  &#x3C;md-tab ng-repeat=&#x22;room in roomsCtrl.rooms&#x22;&#x3E;
    &#x3C;md-tab-label&#x3E;{{room.name}} &#x3C;a ng-click=&#x22;roomsCtrl.removeRoom($index,room)&#x22;&#x3E;
      &#x3C;md-icon md-svg-icon=&#x22;navigation:close&#x22;&#x3E;&#x3C;/md-icon&#x3E;
    &#x3C;/a&#x3E;
    &#x3C;/md-tab-label&#x3E;
    &#x3C;div ui-view flex&#x3E;&#x3C;/div&#x3E;
&#x3C;/md-tabs&#x3E;
&#x3C;/md-content&#x3E;
</code></pre>

Ne feledkezzünk meg kilépéskor leiratkoztatni a szobáról a usert. Ehhez egészítsük ki az account.service.js logout függvényét és vegyük fel az openedRoomsFactory és a socketFactory függőségeket is: 

<pre><code>
var openedRooms = openedRoomsFactory.getRooms();
//logout from rooms
_.forEach(openedRooms, function (room) {
socketFactory.emit("unsubscribe", {room: room._id, user: getLoggedInUser()});
});
</code></pre>

töröljük a megnyitott szobákat is:

<pre><code>
delete $localStorage.rooms;
delete $localStorage.index;
</code></pre>

A routingot is egészítsük ki az index.routes.js fájlban. Itt már létezik egy "rooms" nevű állapot, ezt egészítsük ki, hogy a következőképpen nézzen ki:

<pre><code>
.state('rooms', {
  url: '/rooms',
  templateUrl: 'app/room/room.tabs.tmpl.html',
  controller: 'RoomsController',
  controllerAs: 'roomsCtrl',
  data: {authenticated: true}
});
</code></pre>

A create.room.fab.directive.js-hez adjuk hozzá az openedRoomsFactory függőséget valamint atdjuk meg azt is, hogy ha új szobát hozunk létre, az egyből bekerüljön a megnyitott szobák közé és meg is nyíljon: 

<pre><code>
openedRoomsFactory.addRoom(result);
$state.go("rooms.room",{id: result._id});
</code></pre>
 

### Üzenetek kezelése

#### step_messages

Az üzenetek aspektus számára hozzuk létre a message nevű mappát. 
Üzenet modell: message.model.js fájl létrehozása message mappában

<pre><code>
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
</code></pre>

Hozzunk létre egy service-t, amely az üzenetek kezeléséért lesz felelős! message.service.js message mappában

<pre><code>
(function() {
'use strict';

angular
  .module('angularChat')
  .service('MessageService',MessageService);

/** @ngInject */
function MessageService($resource,apiUrl,Message,Room,AccountService) { 
}
})();
</code></pre>

A Service-ben két függvényt kell megírnunk.

* A szoba üzeneteit lekérő függvény
<pre><code>
function getRoomMessages(roomId){
	return Message.query({id: roomId});
}
</code></pre>

* Új üzenet létrehozása a szobában
<pre><code>
function createRoomMessage(roomId, message){
var newMessage = new Message();
newMessage.text = message;
newMessage.user = AccountService.getLoggedInUser();
newMessage.$save({id: roomId});
}
</code></pre>

A két függvény publikussá tételéért vegyük fel a Service elején a következő utasításokat:

<pre><code>
this.getRoomMessages = getRoomMessages;
this.createRoomMessage = createRoomMessage;
</code></pre>

Hívjuk meg ezt a két függvényt az alkalmazás megfelelő pontjain! 

room.item.controller.js fájlban: 

A service használatához adjuk hozzá a függőségekhez! 
Vegyük fel a két változót: 

<pre><code>
roomItemCtrl.newMessage = "";
roomItemCtrl.messages = MessageService.getRoomMessages(roomId);
</code></pre>

Új üzenet létrehozása: 

<pre><code>
roomItemCtrl.createMessage = function () {
MessageService.createRoomMessage(roomId, roomItemCtrl.newMessage);
roomItemCtrl.newMessage = "";
};
</code></pre>

Sockethez tartozó rész:

<pre><code>
socketFactory.on("new message",function (message) {
roomItemCtrl.messages.push(message);
});
</code></pre>

Nyissuk meg a room/room.item.html template-et!
Készítsünk egy angular material-os listát, melyben a roomItemCtrl.messages változót jelenítjük meg: 

<pre><code>
&#x3C;md-list scroll=&#x22;roomItemCtrl.messages&#x22;&#x3E;
&#x3C;md-subheader class=&#x22;md-info&#x22;&#x3E;Messages in room {{ roomItemCtrl.room.name }}&#x3C;/md-subheader&#x3E;
&#x3C;message ng-repeat=&#x22;message in roomItemCtrl.messages&#x22; message=&#x22;message&#x22;
         author=&#x22;roomItemCtrl.allusers[message.authorId]&#x22;
         ng-class=&#x22;{ &#x27;repeated-author&#x27; : $index&#x3E;0 &#x26;&#x26; message.authorId == roomItemCtrl.messages[$index-1].authorId}&#x22;&#x3E;&#x3C;/message&#x3E;
&#x3C;/md-list&#x3E;
</code></pre>

Megjegyzés: 
•	az ng-class rész az üzenetek megjelenítésének testreszabására van. 

Új üzenet létrehozására: 

<pre><code>
&#x3C;md-input-container class=&#x22;md-accent&#x22;&#x3E;
&#x3C;label&#x3E;New Message&#x3C;/label&#x3E;
&#x3C;input ng-enter=&#x22;roomItemCtrl.createMessage()&#x22; ng-model=&#x22;roomItemCtrl.newMessage&#x22; md-maxlength=&#x22;350&#x22;/&#x3E;
&#x3C;/md-input-container&#x3E;
</code></pre>

Az előzőekben használtuk a message direktívát, de az sehol sem létezik, ezt sajnos nekünk kell megírnunk :)
A message mappában hozzuk létre a message.directive.js fájlt: 

<pre><code>
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
</code></pre>

Túl sok mindent nem csinál, megjeleníti a megadott template-et, ami még nem létezik. Hozzuk létre a hiányzó message.item.tmpl.html fájlt a következő tartalommal:

<pre><code>
&#x3C;md-list-item class=&#x22;contact-item md-2-line selected&#x22;&#x3E;
&#x3C;img ng-src=&#x22;{{ messageCtrl.author.imageUrl }}&#x22; class=&#x22;md-avatar message-avatar&#x22;/&#x3E;
&#x3C;div class=&#x22;md-list-item-text compact&#x22;&#x3E;
  &#x3C;p style=&#x22;text-align: right;&#x22;&#x3E;&#x3C;small&#x3E;&#x3C;strong&#x3E;{{messageCtrl.author.username | username }}&#x3C;/strong&#x3E;, {{ messageCtrl.message | createdDate }}&#x3C;/small&#x3E;&#x3C;/p&#x3E;
  &#x3C;div class=&#x22;message-content&#x22;&#x3E;{{messageCtrl.message.text}}&#x3C;/div&#x3E;
&#x3C;/div&#x3E;
&#x3C;/md-list-item&#x3E;
</code></pre>

Az md-list direktíva elemeit akarjuk megadni, így az md-list-item kötelező. 


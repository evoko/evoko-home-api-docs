# Evoko Home API docs

API documentation for Evoko Home version: `v2.1.x`

- [Introduction](#introduction)
- [Connect to Evoko Home over DDP](#connect)
- [Register a client group - `registerClientGroup2x`](#registerClientGroup2x)
- [Register an integration user - `registerIntegrationUser`](#registerIntegrationUser)
- [Subscribe to collections](#collections)
- [Create a meeting - `createBooking`](#createBooking)
- [Update a meeting - `updateEvent`](#updateEvent)
- [Delete a meeting - `deleteEvent`](#deleteEvent)
- [Check-in a meeting - `confirmMeeting`](#confirmMeeting)
- [Search for available rooms - `getAvailableRoomsAdvancedNew`](#getAvailableRoomsAdvancedNew)
- [Edit room equipment status - `editRoomEquipmentReport`](#editRoomEquipmentReport)

## <a name="introduction"></a> Introduction

Welcome to the Evoko Home Open API!

Throughout this document you will find the available API methods for Evoko Home along with its parameters in a table. You will also find basic requests examples written for [Node.js](https://nodejs.org/) along with some common response objects.

> 👉 **Disclaimer!** The code examples provided in this document are written as basic examples and therefore shouldn't automatically be considered as production ready code, use your own judgement.

Evoko Home is the server side software necessary for using the Evoko Liso devices that sits between (or acts as) your booking system and configuration tool which your Evoko Liso units synchronizes with. The Evoko Home platform is built on the full stack JavaScript framework [Meteor](https://www.meteor.com/) which uses the DDP protocol to send/receive requests.

### Versioning

Evoko Home attempts to follow [semantic versioning](https://semver.org/) (e.g. `v2.0.0`), for you as a developer this means that:

- Breaking API changes can be expected at **major** version bumps (i.e. `v1.x.y`, `v2.x.y`, `v3.x.y`).
- Non-breaking API additions can occur at **minor** version bumps (i.e. `v2.0.x`, `v2.1.x`, `v2.2.x`).

### Evoko provides

- The API itself as part of the Evoko Home server software.
- API documentation on the various API methods and basic usage.
- Technical support for issues with the API. This is limited to issues strictly related to the Evoko Home API or configuration. We cannot help with issues or bugs in integration platforms or third party libraries/solutions.

### Evoko does not provide

- Any programming services for the Evoko Home API.
- Help with finding suitable developers for a Evoko Home API project.

<!-- TODO: Add info on how to submit API feature/improvement requests -->

## <a name="connect"></a> Connect to Evoko Home via DDP

As mentioned in the introduction Evoko Home is built on full stack JavaScript framework [Meteor](https://www.meteor.com/) which uses the DDP protocol to communicate and is not your typical REST API. To connect and send request you will need to use a DDP client - _which one?_ you may ask, that's up to you to decide however in our examples we will use a [ddp client available on npm](https://www.npmjs.com/package/ddp).

To connect, you may use an unencrypted websocket (`ws`) on your unencrypted application port (default `3000`) or use an encrypted websocket (`wss`) on your encrypted application port (default `3002`) over TLS v1.2. For example:

- **Unencrypted:** `ws://localhost:3000/websocket`
- **Encrypted:** `wss://server.domain.tld:3002/websocket`

> 👉 **Note!** With the above linked ddp client we have been unsuccessful in connecting to `localhost` on the encrypted protocol/port (i.e. `wss`/`3002`) so for connecting to an Evoko Home server on `localhost` you may need to use the unencrypted protocol/port (i.e. `ws`/`3000`). For remote servers this should not be an issue in our experience. 

To authenticate, you can use the username/password of any Global admin user that exists in Evoko Home. Another option is to use the pre-generated API user (`defaultDevIntegrationUser`) whos password (aka `groupToken`) can easily be found and copied under "Global Settings" in the Evoko Home web interface (recommended).

### Request example

```js
const DDPClient = require('ddp');
const sha256 = require('sha256');

const USERNAME = 'defaultDevIntegrationUser';
const PASSWORD = 'very-secret-token';
const URL = 'wss://server.domain.tld:3002/websocket';

const PASSWORDHASH = sha256(PASSWORD);

const ddpclient = new DDPClient({
  autoReconnect: true,
  autoReconnectTimer : 500,
  maintainCollections : true,
  ddpVersion : 1,
  url: URL
});

ddpclient.connect(function(error, wasReconnect) {
  if (error) {
    console.log('DDP connection error!');
    return;
  }

  if (wasReconnect) {
    console.log('Reestablishment of a connection.');
  } else {
    console.log('Connected!');
  }

  ddpclient.call('login',
    [{
      user: {username: USERNAME},
      password: {digest: PASSWORDHASH, algorithm: 'sha-256'}
    }],
    function (err, result) {
      if (result) {
        console.log(result);
      } else {
        console.log(err);
      }
    }
});
```

### Response example

Your token is valid for 90 days. The `id` represents your user (in our case `defaultDevIntegrationUser`).

```js
{ id: 'dM7mDamj24k3QnjvF',
  token: '042BVXEZdgFYJ2If8iFWkPfnyGzUsvVbqzEHbBq9PVM',
  tokenExpires: 2019-04-01T12:00:00.757Z,
  type: 'password' }
```

### Error response example

```js
// Invalid or non-existing username
{ isClientSafe: true,
  error: 403,
  reason: 'User not found',
  message: 'User not found [403]',
  errorType: 'Meteor.Error' }

// Invalid password
{ isClientSafe: true,
  error: 403,
  reason: 'Incorrect password',
  message: 'Incorrect password [403]',
  errorType: 'Meteor.Error' }

// User lacks permission
{ isClientSafe: true,
  error: 401,
  reason: 'User is not authorized!',
  message: 'User is not authorized! [401]',
  errorType: 'Meteor.Error' }
```

## <a name="registerClientGroup2x"></a> Register a client group - `registerClientGroup2x`

> 👉 **Note!** For authenticating with your own application, rather than registering your own integration group we recommend using the default API user (`defaultDevIntegrationUser`) and that you skip using this API method.

This method (`registerClientGroup2x`) is used to register a new client group and only have a single parameter which is the name of the group (e.g. `myNewGroup`).

If the request is successful a response object will be received, if it fails it will throw an error.

### Request example

```js
ddpclient.call(
  'registerClientGroup2x',
  ['myNewGroup'],
  function(err, result) {
    if (result) {
      console.log(result);
    } else {
      console.log(err);
    }
  }
);
```

### Response example

```js
{ groupName: 'myNewGroup',
  groupToken: 'zEd_vOJrwo4W3AbFsPPhpgRzacLwY7cfrQIIrxLDC7k' }
```

### Error response example

```js
{ isClientSafe: true,
  error: 'Unable to register client group with that name!',
  message: '[Unable to register client group with that name!]',
  errorType: 'Meteor.Error' }
```


## <a name="registerIntegrationUser"></a> Register an integration user - `registerIntegrationUser`

> 👉 **Note!** For authenticating with your own application, rather than registering your own integration user we recommend using the default API user (`defaultDevIntegrationUser`) and that you skip using this API method.

This method (`registerIntegrationUser`) is used to register a new integration user and only have a single parameter which is `groupToken` for the group the user should be registered to.

If the request is successful a response object will be received, if it fails it will throw an error.

### Request example

```js
ddpclient.call(
  'registerIntegrationUser',
  ['zEd_vOJrwo4W3AbFsPPhpgRzacLwY7cfrQIIrxLDC7k'],
  function(err, result) {
    if (result) {
      console.log(result);
    } else {
      console.log(err);
    }
  }
);
```

### Response example

```js
{ _id: 'QegBzfAgpjMh55Bun',
  createdAt: 2019-01-01T12:00:16.518Z,
  services:
   { password:
      { bcrypt: '$2a$10$0vYLzWlFA2cl//Hn6vO6QeW3yGpMxNNoQuKqbSQY./UzMA24j8jeC' } },
  username: 'deviceBReyXScTsy2yBhcvZ',
  profile:
   { name: 'deviceBReyXScTsy2yBhcvZ',
     pin: '',
     rfid: '',
     type: 'APIIntegration',
     originalToken: null,
     clientGroup: 'myNewGroup',
     groupToken: 'zEd_vOJrwo4W3AbFsPPhpgRzacLwY7cfrQIIrxLDC7k' } }
```

## <a name="collections"></a> Subscribe to collections

Evoko Home stores meetings, users, settings etc in what Meteor/MongoDB calls "collections". A collection can be considered as group or table and within a collection data can be stored sub-groups of JSON formatted "documents", each with its own unique ID.

For example, in Evoko Home each user is stored in their own document (which contains user information as their name, email, PIN, RFID etc) and all users are collectively grouped under a single collection called "users".

To get a better understanding of the Evoko Home database structure you can use a 3rd party tool like [Robo 3T](https://robomongo.org/) which is a MongoDB client that allows you to view the different database collections via a desktop app.

> ⚠️ **Warning!** Adding, deleting or editing anything directly in the database without the use of the specified Evoko Home API methods is strongly discouraged as it may lead to unforeseen consequences and is therefor not supported. When subscribing to a collection do so by a read-only approach.

<!-- TODO: Add more info on how subscribing works -->

### Collections

<!-- TODO: Confirm all keys/parameters in below table and add additional info -->

| Name              | Collection key     | Parameters        | Comment |
| ----------------- | -------------------| ------------------| --------|
| `bookings`        | `allBookings`      | `roomId` or `ALL` | Contains all current meetings. Note that only meetings 7 days into the future is synced to this collection from external booking system (e.g. Office 365). |
| `rooms`           | `allRooms`         | `roomId` or `ALL` | Contains all rooms added in Evoko Home. |
| `users`           | `users`            |                   | Contains all users added in Evoko Home. |
| `settings`        | `specificSettings` | `roomId`          | Contains all settings profiles that are applied to rooms. |
| `structures_flat` | `structuresFlat`   |                   | Contains the room structures (i.e. Country, City, Building, Floor) and maps which rooms that are grouped under which structures. |
| `customEquipment` | `customEquipment`  |                   | Contains documents of any eventual custom room equipment that has been added in Evoko Home. |

### Request example

```js
// Subscribes to the room with the roomId "MBPMGDaomwnHawrt6"
ddpclient.subscribe(
  'allRooms',
  ['MBPMGDaomwnHawrt6'],
  function() {
    console.log(ddpclient.collections.rooms);
  }
);
```

### Response example

```js
{ MBPMGDaomwnHawrt6:
   { _id: 'MBPMGDaomwnHawrt6',
     name: 'Fika room',
     mail: 'fika-room@domain.tld',
     address: 'fika-room@domain.tld',
     id: 'Fika room',
     numberOfSeats: 4,
     alias: 'Fika room',
     isActive: true,
     isDeleted: false,
     equipment:
      { lights: true,
        projector: null,
        computer: null,
        teleConference: null,
        wifi: true,
        whiteboard: null,
        videoConference: null,
        display: null,
        minto: true,
        ac: true,
        information: null },
     structureId: 'xSEQpnFEZ2KFWroBo',
     userIds: [],
     assigned: true } }
```

## <a name="createBooking"></a> Create a meeting - `createBooking`

This method (`createBooking`) is used to create a new meeting.

If the request is successful a response object will be received, if it fails it will throw an error.

### Parameters

| Name        | Type    | Mandatory | Comment |
| ----------- | ------- | --------- | ------- |
| `roomId`    | String  | Yes       | The document id of the room (same as the `_id` string). |
| `startDate` | String  | Yes       | Meeting start date in ISO 8601 format (`YYYY-MM-DDTHH:mm:ssZ`). |
| `endDate`   | String  | Yes       | Meeting end date in ISO 8601 format (`YYYY-MM-DDTHH:mm:ssZ`). The **minutes** in this property **must** be set in even quarters (i.e. either `00`, `15`, `30` or `45`). |
| `subject`   | String  | No        | The meetings subject. If left empty or excluded the subject will use the Liso default (`Booked on screen`). |
| `confirmed` | Boolean | No        | `true` considers a meeting as checked-in and `false` the opposite. If excluded from the request, then it will default to `false` for future meetings and `true` for instant meetings (i.e normal Liso behavior). |
| `pin`       | Number  | No\*      | PIN number associated with the user in Evoko Home. Required if authentication for instant meetings (`bookMeetingSettings.auth`) and/or future meetings (`bookFutureMeetingSettings.auth`) is enabled (`true`). If authentication is disabled (`false`), then this value can be set to `null` or the parameter be excluded. |
| `rfid`      | String  | No\*      | RFID associated with the user in Evoko Home. Can be used instead of `pin` if RFID authentication is enabled, otherwise the value can be set to `null` or the parameter be excluded. |
| `metadata`  | Object  | No        | Contains one boolean parameter `demoMode` which should be set to `false`. |

### Request example

```js
ddpclient.call(
  'createBooking',
  [
    {
      roomId: 'MBPMGDaomwnHawrt6',
      startDate: '2019-01-01T15:00:00:00+00:00',
      endDate: '2019-01-01T15:30:00+00:00',
      subject: 'Fika meeting!',
      confirmed: false,
      pin: 9999,
      rfid: null
    }
  ],
  function(err, result) {
    if (result) {
      console.log(result);
    } else {
      console.log(err);
    }
  }
);
```

### Response example

The `id` is the newly created meetings document identifier (i.e `_id` / `bookingId`).

```js
{ type: 'success',
  message: 'MeetingSuccessCreated',
  body: { id: 'FoyY5889YsSFL3rj4' } }
```

### Error response example

```js
// A meeting already exists
{ isClientSafe: true,
  error: 400,
  reason: 'MeetingSuccessExists',
  message: 'MeetingSuccessExists [400]',
  errorType: 'Meteor.Error' }
```

## <a name="updateEvent"></a> Update a meeting - `updateEvent`

This method (`updateEvent`) is used to update existing meetings.

If the request is successful a response object will be received, if it fails it will throw an error.

### Parameters

<!-- TODO: Verify the "ongoing" parameter and add additional info -->

| Name        | Type    | Mandatory | Comment |
| ----------- | ------- | --------- | ------- |
| `bookingId` | String  | Yes       | The document id of the meeting (same as the `_id` string). |
| `id`        | String  | Yes       | The id of the meeting in the integrated booking system (e.g. Office 365). |
| `startDate` | String  | Yes       | Meeting start date in ISO 8601 format (`YYYY-MM-DDTHH:mm:ssZ`). |
| `endDate`   | String  | Yes       | Meeting end date in ISO 8601 format (`YYYY-MM-DDTHH:mm:ssZ`). The **minutes** in this property **must** be set in even quarters (i.e. either `00`, `15`, `30` or `45`). |
| `pin`       | Number  | No\*      | PIN number associated with the user in Evoko Home. Required if authentication for extending ongoing meetings (`extendOngoingMeetingSettings.auth`) and/or extending future meetings (`extendFutureMeetingSettings.auth`) is enabled (`true`). If authentication is disabled (`false`), then this value can be set to `null` or the parameter be excluded. |
| `rfid`      | String  | No\*      | RFID associated with the user in Evoko Home. Can be used instead of `pin` if RFID authentication is enabled, otherwise the value can be set to `null` or the parameter be excluded. |
| `ongoing`   | Boolean | No        | Will be `true` if the meeting is in progress, otherwise `false`. |
| `metadata`  | Object  | No        | Contains one boolean parameter `demoMode` which should be set to `false`. |

### Request example

```js
ddpclient.call(
  'updateEvent',
  [
    {
      bookingId: 'FoyY5889YsSFL3rj4',
      id: 'AAMkADNjMjQ3MTBiLWU2YjYtNDExNi1hMmQzLWFiNDhkY2RjZjg1NwBGAAAAAACk04VXQlknRLAVB3vQZOksBwAy7N4vuvaDT4Acs1TT1y6sAAAAAAENAAAy7N4vuvaDT4Acs1TT1y6sAADktGXSAAA=',
      startDate: '2019-01-01T15:00:00+00:00',
      endDate: '2019-01-01T15:45:00+00:00',
      pin: null,
      rfid: null,
      ongoing: false
    }
  ],
  function(err, result) {
    if (result) {
      console.log(result);
    } else {
      console.log(err);
    }
  }
);
```

### Response example

```js
{ type: 'success',
  message: 'MeetingSuccessUpdate',
  body: null }
```

### Error response example

```js
// Unable to extend meeting e.g due conflict with another existing meeting
{ isClientSafe: true,
  error: 400,
  reason: 'MeetingInvalidExtend',
  message: 'MeetingInvalidExtend [400]',
  errorType: 'Meteor.Error' }
```

## <a name="deleteEvent"></a> Delete a meeting - `deleteEvent`

This method (`deleteEvent`) is used to delete existing meetings.

If the request is successful **no** response object will be received, if it fails it will throw an error.

### Parameters

<!-- TODO: Verify the "ongoing" parameter and add additional info -->

| Name        | Type    | Mandatory | Comment |
| ----------- | ------- | --------- | ------- |
| `bookingId` | String  | Yes       | The document id of the meeting (same as the `_id` string). |
| `pin`       | Number  | No\*      | PIN number associated with the user in Evoko Home. Required if authentication for extending ongoing meetings (`endOngoingMeetingSettings.auth`) and/or extending future meetings (`endFutureMeetingSettings.auth`) is enabled (`true`). If authentication is disabled (`false`), then this value can be set to `null` or the parameter be excluded. |
| `rfid`      | String  | No\*      | RFID associated with the user in Evoko Home. Can be used instead of `pin` if RFID authentication is enabled, otherwise the value can be set to `null` or the parameter be excluded. |
| `ongoing`   | Boolean | No        | Will be `true` if the meeting is in progress, otherwise `false`. |
| `metadata`  | Object  | No        | Contains one boolean parameter `demoMode` which should be set to `false`. |

### Request example

```js
ddpclient.call(
  'deleteEvent',
  [
    {
      bookingId: 'FoyY5889YsSFL3rj4',
      pin: 9999,
      rfid: null,
      ongoing: true
    }
  ],
  function(err, result) {
    if (result) {
      console.log(result);
    } else {
      console.log(err);
    }
  }
);
```

## <a name="confirmMeeting"></a> Check-in a meeting - `confirmMeeting`

This method (`confirmMeeting`) is used to check-in an existing meeting.

If the request is successful a response object will be received, if it fails it will throw an error.

### Parameters

| Name        | Type   | Mandatory | Comment |
| ----------- | ------ | --------- | ------- |
| `bookingId` | String | Yes       | The document id of the meeting (same as the `_id` string). |
| `pin`       | Number | No\*      | PIN number associated with the user in Evoko Home. Required if authentication for check-in (`confirmMeetingSettings.auth`) is enabled (`true`). If authentication is disabled, then this value can be set to `null` or the parameter be excluded. |
| `rfid`      | String | No\*      | RFID associated with the user in Evoko Home. Can be used instead of `pin` if RFID authentication is enabled, otherwise the value can be set to `null` or the parameter be excluded.|
| `metadata`  | Object | No        | Contains one boolean parameter `demoMode` which should be set to `false`. |

### Request example

```js
ddpclient.call(
  'confirmMeeting',
  [
    {
      bookingId: 'QGnHDQDdZ3FhAXpv3',
      pin: null,
      rfid: null
    }
  ],
  function(err, result) {
    if (result) {
      console.log(result);
    } else {
      console.log(err);
    }
  }
);
```

### Response example

```js
{ type: 'success',
  message: 'Booking is successfully confirmed!',
  body: null }
```

### Error response example

```js
// No PIN provided
{ isClientSafe: true,
  error: 400,
  reason: 'PinIsRequired',
  message: 'PinIsRequired [400]',
  errorType: 'Meteor.Error' }

// Invalid PIN
{ isClientSafe: true,
  error: 400,
  reason: 'PinWrongNumber',
  message: 'PinWrongNumber [400]',
  errorType: 'Meteor.Error' }

// Invalid RFID
{ isClientSafe: true,
  error: 400,
  reason: 'RFIDWrongNumber',
  message: 'RFIDWrongNumber [400]',
  errorType: 'Meteor.Error' }
```

## <a name="getAvailableRoomsAdvancedNew"></a> Search for available rooms - `getAvailableRoomsAdvancedNew`

This method (`getAvailableRoomsAdvancedNew`) is used to get available rooms based on a time interval and room properties such as location, capacity and equipment.

If the request is successful a response object will be received which contains two arrays (`fullMatchArray` and `partialMatchArray`), if it fails it will throw an error.

The first array (`fullMatchArray`) contain rooms that fully match the conditions, the second array (`partialMatchArray`) contain rooms that partially match the conditions. Rooms that fail at least 1 condition but does not fail more than 3 will be returned as a partial match (with the exception of location and time interval which will filter results regardless).

### Parameters

| Name              | Type    | Mandatory | Comment |
| ----------------- | ------- | --------- | ------- |
| `room`            | Object  | Yes       | This parameter is mainly by the Liso to declare which device search is executed from. We recommend you set the value to `{_id: 'not-a-liso'}`. |
| `startDate`       | String  | Yes       | Start date in ISO 8601 format (`YYYY-MM-DDTHH:mm:ssZ`). |
| `endDate`         | String  | Yes       | End date in ISO 8601 format (`YYYY-MM-DDTHH:mm:ssZ`). |
| `location`        | Object  | Yes       | Object with the following parameters `country`, `city`, `building` and `floor`. The value can be set to each respective `structureId` or `false` to filter results. |
| `seats`           | String  | Yes       | Available values are `1-5`, `5-10`, `10-20`, `20-50`, `50+` to filter rooms based on their seat capacity. |
| `equipment`       | Object  | Yes       | Object with the following parameters `lights`, `projector`, `computer`, `teleConference`, `wifi`, `whiteboard`, `videoConference`, `display`, `minto`, `ac` and `information`. The value for these parameters are boolean, set to `true` to return rooms with this equipment and set to `false` (or remove the parameter) to not include in the search filter. |
| `customEquipment` | Array   | Yes       | The array can be left empty (e.g. `[]`) which will ignore any custom equipment when filtering. To filter based on some custom equipment item, include the object (e.g. `[{_id: '8X8r9q2KSQAh8M9Lw', name: 'Fika', isChecked: true}]`) and set `isChecked` to `true`. |
| `metadata`        | Object  | Yes       | Contains one boolean parameter `demoMode` which should be set to `false`. |

### Request example

```js
ddpclient.call(
  'getAvailableRoomsAdvancedNew',
  [
    {
      room: { _id: 'not-a-liso' },
      startDate: '2019-01-01T12:00:00+00:00',
      endTDate: '2019-01-01T12:30:00+00:00',
      location: { country: false, city: false, building: false, floor: false },
      seats: '1-5',
      equipment: {
        wifi: false,
        whiteboard: false,
        videoConference: false,
        computer: false,
        projector: false,
        teleConference: false,
        information: false,
        minto: false,
        display: false,
        lights: false,
        ac: false
      },
      customEquipment: [],
      isFiltered: true,
      metadata: { demoMode: false }
    }
  ],
  function(err, result) {
    if (result) {
      console.log(result);
    } else {
      console.log(err);
    }
  }
);
```

### Response example

```js
{ fullMatchArray:
   [ { _id: 'MBPMGDaomwnHawrt6',
       name: 'Fika room',
       alias: 'Fika room',
       mail: 'fika-room@domain.tld',
       address: 'fika-room@domain.tld',
       id: 'Fika room',
       isActive: true,
       isDeleted: false,
       numberOfSeats: 4,
       equipment: [Object],
       customEquipment: [Array],
       structureId: 'xSEQpnFEZ2KFWroBo',
       userIds: [],
       assigned: true } ],
  partialMatchArray:
   [ { _id: '7Ag8arosM5DW5KtCs',
       name: 'Another fika room',
       alias: 'Another fika room',
       mail: 'another-fika-room@domain.tld',
       address: 'another-fika-room@domain.tld',
       id: 'Another fika room',
       isActive: true,
       isDeleted: false,
       numberOfSeats: 8,
       equipment: [Object],
       customEquipment: [Array],
       structureId: 'vj69HgSWyN5GhRZ5H',
       userIds: [],
       assigned: true } ] }
```

## <a name="editRoomEquipmentReport"></a> Edit room equipment status - `editRoomEquipmentReport`

This method (`editRoomEquipmentReport`) can be used to edit the status of a rooms equipment as working/fixed (`true`), broken (`false`) or disabled (`null`).

If the request is successful a response object will be received, if it fails it will throw an error.

### Parameters

| Name              | Type    | Mandatory | Comment |
| ----------------- | ------- | --------- | ------- |
| `roomObject`      | Object  | Yes       | Object containing the following 3 parameters `_id`, `equipment` and `customEquipment` for the room which equipment you wish to edit. |
| `adminReportFlag` | Boolean | No        | Determines whether admin authentication is required (`true`) or not (`false`) when reporting equipment. |
| `pin`             | Number  | No\*      | PIN number associated with the user in Evoko Home. Required if authentication for reporting equipment (`reportSettings.auth`) is enabled (`true`). If authentication is disabled, then this value can be set to `null` or the parameter be excluded. |
| `rfid`            | String  | No\*      | RFID associated with the user in Evoko Home. Can be used instead of `pin` if RFID authentication is enabled, otherwise the value can be set to `null` or the parameter be excluded. |
| `metadata`        | Object  | Yes       | Contains one boolean parameter `demoMode` which should be set to `false`. |

### Request example

```js
ddpclient.call(
  'editRoomEquipmentReport',
  [
    {
      roomObject: {
        _id: 'MBPMGDaomwnHawrt6',
        equipment: {
          wifi: true,
          whiteboard: true,
          videoConference: true,
          computer: false,
          projector: null,
          teleConference: true,
          information: null,
          minto: true,
          display: null,
          lights: true,
          ac: null
        },
        customEquipment: [
          {
            _id: '8X8r9q2KSQAh8M9Lw',
            name: 'Fika',
            isChecked: true
          },
          {
            _id: 'uStt5CX9CDpKapaCX',
            name: 'More fika',
            isChecked: null
          }
        ]
      },
      adminReportFlag: false,
      pin: null,
      rfid: null,
      metadata: { demoMode: false }
    }
  ],
  function(err, result) {
    if (result) {
      console.log(result);
    } else {
      console.log(err);
    }
  }
);
```

### Response example

```js
// When reported as fixed
{ type: 'success',
  message: 'RoomSuccessEquipUpdateRepaired',
  body: null }

// When reported as broken
{ type: 'success',
  message: 'RoomSuccessEquipUpdateBroken',
  body: null }
```

### Error response example

```js
// Non-admin attepts to report stuff as fixed when adminReportFlag = true
{ isClientSafe: true,
  error: 403,
  reason: 'NoAuthorizedResourceAccess',
  message: 'NoAuthorizedResourceAccess [403]',
  errorType: 'Meteor.Error' }
```

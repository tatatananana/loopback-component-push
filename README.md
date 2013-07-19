# Loopback Push Notification

Loopback Push Notification is a set of server side functions to enable mobile push notification services. It consists of
the following components:

## Features

* DeviceRegistration model and APIs to send notifications to devices of interest

* Wrapper APIs to multiple mobile push notification platforms, including:
  * apns
  * c2dm/gcm (TBD)
  * mpns (TBD)

* Scheduling notifications (TBD)

## Device Registration

The mobile applications need to first register itself with the backend using DeviceRegistration model and APIs.

### Model
A record of DeviceRegistration has the following properties:

    id: String (automatically generated id to identify the record)
    appId: String, (application id registerd with the Application model)
    appVersion: String, (optional application version)
    userId: String,
    deviceToken: String,
    deviceType: String,
    subscriptions: [String],
    status: String,
    created: Date,
    modified: Date

### Register a new device
To regsiter a device, we can call the DeviceRegistration.create API as follows:

    DeviceRegistration.create({
        appId: 'MyLoopbackApp',
        userId: 'raymond',
        deviceToken: '75624450 3c9f95b4 9d7ff821 20dc193c a1e3a7cb 56f60c2e f2a19241 e8f33305',
        deviceType: 'apns',
        created: new Date(),
        modified: new Date(),
        status: 'Active'
    }, function (err, result) {
        console.log('Registration record is created: ', result);
    });

The DeviceRegistrtion model is exposed as CRUD REST APIs via loopback.

        POST http://localhost:3000/deviceRegistrations
        {
            "appId": "MyLoopbackApp",
            "userId": "raymond",
            "deviceToken": "75624450 3c9f95b4 9d7ff821 20dc193c a1e3a7cb 56f60c2e f2a19241 e8f33305",
            "deviceType": "apns"
        }

### Send notifications

    /**
     * Push notification to a given device
     * @param deviceToken
     * @param notification
     */
    PushService.prototype.pushNotification = function (deviceToken, notification)

    /**
     * Push notification based the application
     * @param appId
     * @param appVersion
     * @param notification
     */
    PushService.prototype.pushNotificationByApp = function (appId, appVersion, notification, cb)

    /**
     * Push notification based the user
     * @param userId
     * @param notification
     */
    PushService.prototype.pushNotificationByUser = function (userId, notification, cb)

## Samples

We embed a folk of [apnagent-ios](https://github.com/logicalparadox/apnagent-ios) under example/apnagent-ios as the
iOS sample app to test push notifications.

### Sample 1


    var apn = require('apn');
    var path = require('path');

    var options = {
        "gateway": "gateway.sandbox.push.apple.com",
        "cert": path.join(__dirname, "credentials/apns_cert_dev.pem"),
        "key": path.join(__dirname, "credentials/apns_key_dev.pem")
    };

    var feedbackOptions = {
        "gateway": 'feedback.sandbox.push.apple.com',
        "cert": path.join(__dirname, "credentials/apns_cert_dev.pem"),
        "key": path.join(__dirname, "credentials/apns_key_dev.pem"),
        "batchFeedback": true,
        "interval": 300
    }


    var apnConnection = new apn.Connection(options);
    apnConnection.on('error', function (err) {
        console.error(err);
    });

    var token = "75624450 3c9f95b4 9d7ff821 20dc193c a1e3a7cb 56f60c2e f2a19241 e8f33305";

    var myDevice = new apn.Device(token);

    var note = new apn.Notification();

    note.expiry = Math.floor(Date.now() / 1000) + 3600; // Expires 1 hour from now.
    note.badge = 3;
    note.sound = "ping.aiff";
    note.alert = "\uD83D\uDCE7 \u2709 You have a new message";
    note.payload = {'messageFrom': 'Caroline'};

    apnConnection.pushNotification(note, myDevice);

## References

1. https://github.com/argon/node-apn
2. https://github.com/rs/pushd
3. https://github.com/logicalparadox/apnagent-ios
4. https://blog.engineyard.com/2013/developing-ios-push-notifications-nodejs


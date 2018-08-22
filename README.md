# How to prepare service termination with Firebase Remote Config

The background of why I will write my first tutorial for this
  - Actually the service created will be terminated
  - All other Engineers left except me as a client side Engineer

## Purpose

  - Not much coding and screen will be shared here, just hope to be as general as possible
  - For how to start, connect and config your Google Firebase service, please find help in here, https://firebase.google.com/

### How to start
Emotionally service termination is one of the most upset feature to be done as a developer.  Normally it should be worked like this

> When you press a button or command to turn off the server,
> the connection to your service will be terminated.
> A bunch of code at the very beginning will have a branch that
> showing if the service is not connected, then showing the termination screen.

For my case, since the server Engineer has gone at once.  I am not eligible to touch the server so this is a workaround which should be rather independent that can be used for anyone

### Tech

What you need is a Firebase project, and config the parameters required in the Remote Config session

* is_service_end - Flags if service ended or not, what I would suggest that "no" or "countdown" (if necessary) and other text or even empty for service termination
* service_countdown - The countdown time for your display, suggested to be in a proper format which can be used for any purpose.  e.g. 2018-09-30T08:00:00+0000

### Code Snippet

This part I will use Swift 4 as an example.  The first step is to connect to Firebase Remote Config

```sh
    let config = RemoteConfig.remoteConfig()
```

It is that simple, of course you have to setup the Firebase authenticaton before.  Please take a look here for more information.  [Swift] https://firebase.google.com/docs/ios/setup?hl=ja

```sh
        config.fetch(withExpirationDuration: 0) { status, error in
            if let error = error {
                print(error.localizedDescription)
                return endState(.end)
            }
            
            if status == .success {
                config.activateFetched()
            }
            
            guard let serviceEndStatus = config.configValue(forKey: "service_end").stringValue else { return }
            
            if serviceEndStatus == "no" {
                return endState(.notyet)
            } else if serviceEndStatus == "countdown" {
                guard let isoDate = config.configValue(forKey: "service_countdown").stringValue else {
                    return endState(.countdown)
                }
                
                let dateFormatter = DateFormatter()
                dateFormatter.dateFormat = "yyyy-MM-dd'T'HH:mm:ssZ"
                dateFormatter.locale = Locale(identifier: "en_US_POSIX")
                dateFormatter.timeZone = TimeZone(identifier: "Asia/Tokyo")

                guard let endDate = dateFormatter.date(from: isoDate) else {
                    return endState(.countdown)
                }
                
                let calendar = Calendar.current
                let start = calendar.startOfDay(for: Date())
                let end = calendar.startOfDay(for: endDate)
                
                let components = calendar.dateComponents([.day], from: start, to: end)
                if let remainDay = components.day {
                    self.serviceEndPeriod = remainDay
                }
                
                return endState(.countdown)
            } else {
                return endState(.end)
            }
        }
```
I have prepared what you may need to use here

* You may have to calculate the remaining day of the project, which you can set it for your countdown period use
* You may add your state as you need.  What is necessary for me is just a countdown phase to show user how long they can use the application, and a thank you screen once service stopped.

### Points to note finally
* Since I cannot check on the case even if the Firebase Project removed, I would suggest you can make a simple flag to force moving to the notification screen
* [TODO] If possible I will add some more screenshot as an improvement later

**Thank you for reading this upset tutorial**


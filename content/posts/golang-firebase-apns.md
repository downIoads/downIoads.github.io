---
title: "How to use APNs with Golang and Google Firebase + Firestore"
date: 2024-01-01T20:00:00+02:00
description: Retrieve data from Firestore and send out push notifications with this simple code.
draft: false
tags: [iOS, go, programming, firebase, tutorial]
---

## Introduction

If you are building a SwiftUI app like me, you will probably encounter a situation where you want to be able to send out Push Notifications to your users. Google Firebase offers a free plan (completely free with pretty generous usage) that allows you to do just that. Combined with Google Firestore you even get a small database where you can store device tokens (your users will write their device token to the database by using your app, and you send out notifications to all device tokens). This post is not about the Google Firebase / Firestore setup, but here is a note: If you directly go to the Firestore website and try to create an account it will force you to give them your payment info even if you choose the free plan. But if you just create a Firebase account with the Spark plan (free), you will be able to access Firestore from your dashboard without ever giving out your payment information. This can be useful as you might make a mistake in your Firestore Rules setup which allows anyone to write to it, which potentially could lead to very large bills. But if you never gave Google your payment info, you do not have to worry about scenarios such as this one. So let's look at the Golang code I use to send out APNs notifications to my users.

## Go

```go

package main

import (
    "context"
    "log"       // logging
    "time"      // timeouts
    firebase "firebase.google.com/go"
    "firebase.google.com/go/messaging"
    "google.golang.org/api/iterator"
    "google.golang.org/api/option"
)


func main() {
    sendNotifications("test notification")
}


func sendNotifications(notificationText string) {
    // ---- Connect to Firebase ----
    ctx := context.Background()
    sa := option.WithCredentialsFile("./firebaseAdmin.json")	// you create and dl this from your firebase dashboard
    app, err := firebase.NewApp(ctx, nil, sa)
    if err != nil {
      log.Fatalln(err)
    }
    client, err := app.Firestore(ctx)
    if err != nil {
      log.Fatalln(err)
    }
    defer client.Close()

    // ---- Preparations ----
    curTime := int(time.Now().Unix())
    deviceTokenList := []string{}

    // ---- Read data from firestore collection (in my case it is called device-tokens-apns) ----
    iter := client.Collection("device-tokens-apns").Documents(ctx)
    for {
            doc, err := iter.Next()
            if err == iterator.Done {
                    break
            }
            if err != nil {
                    log.Fatalf("Failed to iterate: %v", err)
            }
            
            // in my case each entry is a dict with keys "timestamp" and "token"

            // convert timestamp to int
            timestampInterface, ok := doc.Data()["timestamp"]
            if !ok {
                log.Fatal("Failed to convert timestamp to int")
            }
            //      first from the interface{} get the int64
            timestampInt64, _ := timestampInterface.(int64)
            //      then cast int64 to int
            timestamp := int(timestampInt64)

            // convert token to string
            tokenInterface, ok := doc.Data()["token"]
            if !ok {
                log.Fatal("Failed to convert token to string")
            }
            //      cast interface{} to string
            tokenString, _ := tokenInterface.(string)

            //fmt.Println("Token:", tokenString)
            //fmt.Println("Timestamp:", timestamp)

            // if timestamp less old than 1 month, add the token to the list (dont notify dead devices, each app start renews token)
            if curTime - timestamp < 2592000 {
                // only add string to slice if its not already in there
                isNewString := true
                for _, s := range deviceTokenList {
                    if s == tokenString {
                        isNewString = false
                    }
                }

                if isNewString {
                    deviceTokenList = append(deviceTokenList, tokenString)  
                }

            }

    }

    // ---- Send notification to each device in token list ----
    fcmClient, err := app.Messaging(ctx)
    if err != nil {
            log.Fatalf("error getting Messaging client: %v\n", err)
    }
    // note that the multi-send-notification is decprecated so my approach is recommended:
    for _, registrationToken := range deviceTokenList {
        // fmt.Println("Now trying to send notification to:", registrationToken)

        // define notification message (IMPORTANT: this code works, but the code in the Google Documentation for APNs did NOT work for me)
        message := &messaging.Message{
                    Notification: &messaging.Notification{
                        Title: "My notification title",
                        Body: notificationText,
                    },
                    Token: registrationToken,
        }

        // send notification to device (ignore return, if there is a problem with the token just don't send a notification to it)
        _, _ = fcmClient.Send(ctx, message)
        
    }

}


```

## Conclusion
It was frustrating for me that the [Google documentation](https://firebase.google.com/docs/cloud-messaging/send-message#go) shows Go code that did not send out notifications for me. There are no errors, but also no notifications are being sent out. The problem fix for me was to replace the "Data" field of messaging.Message with the "Notification" construct I use, which contains "Title" and "Body". All in all, I am glad that Google offers a free plan that works well when set up. It slightly disappoints me that Apple does not offer any service for this (almost every iOS/macOS dev at some point will have to become a Google / Amazon customer for this) but maybe this will change in the future. It just seems to me that this is a very fundamental feature most apps want to make use of and Apple is losing customers for no reason (if you charge $100 a year, you could at least include some way to store device tokens and simplify the sending out notifications process).
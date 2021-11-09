## What is the Meetings API? 
Meetings API allows you to easily integrate real-time, high-quality interactive video meetings into your web apps.  

## Requirements 

- **Vonage Developer Account**: if you don’t have a Vonage account yet, you can get one [here](https://dashboard.nexmo.com/). 
- **API Key and Secret**: Once you’re logged in, you'll find your API Key and Secret in your dashboard under "Getting Started." 

## Basic Terminology

- **Room**: the virtual space in which the video meeting takes place
- **Owner**: usually the creator of the room; this user has special admin capabilities
- **Chat**: space for sending written messages that are visible to all attendees in the room
- **Guest URL**: meeting room URL used by the guest 
- **Host URL**: meeting room URL with additional capabilities used by the session host  
- **Session**: the duration in which there are participants present in the room, from the first participant to join, until the last to leave.  

### Room Types 

#### Instant

An instant room is created for a meeting happening now. 
This room lives for **ten (10) minutes** until the first participant joins the room. 
Once the last participant leaves the room, the room lives for ten more minutes, after which it is deleted. 

#### Long Term

A long term room remains alive until expiration date (max five years). It is typically linked to a recurring meeting, person, or resource. 
It requires an expiration date (in UTC format), and has the option of automatically deleting the room ten minutes after the last participant leaves the room. 
Note that once a room that has been deleted, it will be set to `is_available` = false. 

## Rooms

### Default Room Creation (Instant)

The (POST) endpoint for creating a meeting room: 

```https://api.vonage.com/beta/meetings/rooms```


#### Required Headers

- **Content-Type**: application/json
- **Authorization**: [Basic Authentication](https://developer.nexmo.com/concepts/guides/authentication) with API Key and Secret: 
```Authorization: Basic base64(API_KEY:API_SECRET)```

#### Body 

- `display_name`: (required) the name of the meeting room 
- `metadata`: metadata that will be included in all [callbacks](#Callbacks)
- `type`: `instant` or `long_term`. 
- `expires_at`: (required for `long_term` type) room expiration date in UTC Format
- `recording_options`: object containing various meeting recording options: 
    - `auto_record` (boolean): sets whether the session will be recorded 

#### Request 

```
curl --request POST 'https://api-eu.vonage.com/beta/meetings/rooms' \
--header 'authorization: basic YWFhMDEyOmFiYzEyMzQ1Njc4OQ==' \
--header 'content-type: application/json' \
--data-raw '{
   "display_name":"New Meeting Room"
}'
```


### Response 

Notice that an **instant room** has been created by default, which means that the expiration date was automatically set to 10 minutes after room creation. As this room has not yet expired, `is_available` is set to true.  By default, `auto_record` was set to false. 

```
{
   "id":"a66e451f-794c-460a-b95a-cd60f5dbdc1a",
   "display_name":"New Meeting Room",
   "metadata":null,
   "type":"instant",
   "expires_at":"2021-10-19T17:54:17.219Z",
   "recording_options":{
      "auto_record":false
   },
   "meeting_code":"982515622",
   "_links":{
      "host_url":{
         "href":"https://meetings.vonage.com/?room_token=982515622&participant_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6IjYyNjdkNGE5LTlmMTctNGVkYi05MzBmLTJlY2FmMThjODdj3BK7.eyJwYXJ0aWNpcGFudElkIjoiODNjNjQxNTQtYWJjOC00NTBkLTk1MmYtY2U4MWRmYWZiZDNkIiwiaWF0IjoxNjM0NjY1NDU3fQ.PmNtAWw5o4QtGiyQB0QVeq_qcl6fs0buGMx5t4Fy43c"
      },
      "guest_url":{
         "href":"https://meetings.vonage.com/982515622"
      }
   },
   "created_at":"2021-10-19T17:44:17.220Z",
   "is_available":true
}
```

### Long Term Room Creation 

#### Request 

We will create a long term room that expires on October 21st of 2022, for which each session will be recorded 

```
curl --request POST 'https://api-eu.vonage.com/beta/meetings/rooms' \
--header 'authorization: basic YWFhMDEyOmFiYzEyMzQ1Njc4OQ==' \
--header 'content-type: application/json' \
--data-raw '{
   "display_name":"New Meeting Room",
   "type":"long_term",
   "expires_at":"2022-10-21T18:45:50.901Z", 
   "recording_options": {
       "auto_record": true}
}'
```

#### Response 

```
{
    "id": "bc41d742-b336-4bf6-8643-aa97b5f5025c",
    "display_name": "New Meeting Room",
    "metadata": null,
    "type": "long_term",
    "expires_at": "2022-10-21T18:45:50.901Z",
    "recording_options": {
        "auto_record": true
    },
    "meeting_code": "117744699",
    "_links": {
        "host_url": {
            "href": "https://meetings.vonage.com/?room_token=117744699&participant_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6IjYyNjdkNGE5LTlmMTctNGVkYi05MzBmLTJlY2FmMThjODdjOSJ9.eyJwYXJ0aWNpcGFudElkIjoiZmVlNDVmMDItMDhmOC00ZTdmLWE1MjAtZmYwYjYyZGI2NWM3IiwiaWF0IjoxNjM0NjY3NzQ1fQ.CDHtC3nW2B_jIXhfRTPzznH1j7kzcH3-gbL5h9bxIEE"
        },
        "guest_url": {
            "href": "https://meetings.vonage.com/117744699"
        }
    },
    "created_at": "2021-10-19T18:22:24.965Z",
    "is_available": true
}
```

### Room Management 

#### Individual Room Retrieval 

Notice the ID received in the response. This the ID of the room which will be used for room retrieval. 

```
curl --request GET 'https://api-eu.vonage.com/beta/meetings/rooms/b731a3a9-5552-410b-8d5e-72eac07cb45d' \
--header 'authorization: basic YWFhMDEyOmFiYzEyMzQ1Njc4OQ==' \
--header 'content-type: application/json' 
```
The response will be identical to that of the room creation. 

#### Retrieve all rooms 

To retrieve all rooms, omit the meeting room ID. 

```
curl --location --request GET 'https://api-eu.vonage.com/beta/meetings/rooms/' \
--header 'authorization: basic YWFhMDEyOmFiYzEyMzQ1Njc4OQ==' \
--header 'content-type: application/json'
```

#### Room Deletion 

Use the room ID to perform a DELETE action on a room. 

```
curl --location --request DELETE 'https://api-eu.vonage.com/beta/meetings/rooms/b307d837-c0ce-4619-8c5c-70e418ef9693' \
--header 'Authorization: Basic YWFhMDEyOmFiYzEyMzQ1Njc4OQ==' \
--header 'Content-Type: application/json'
```

This operation will return a 204 status. 

#### Room Update 

A room can be updated by using a PATCH action and the room ID.  Changes can be for `display_name`, `metadata`, `expires_at`, and `recording_options`, and should be included in an object called `update_details`: 

```
curl --location --request PATCH 'https://api-eu.vonage.com/beta/meetings/rooms/e6acf820-7093-416f-a2bd-bcdacd7cc593' \
--header 'Authorization: Basic MTFmMWI4NGY6UnVpbnJIMWxneGZXNGJibQ==' \
--header 'Content-Type: application/json' \
--data-raw '{
  "update_details": {
    "expires_at": "2021-11-11T16:00:00.000Z"
  }
}'
```

## Callbacks 

API meetings callbacks allows you to receive information about session events, participant activity, recording details, and room expiration. To register for callbacks, please be in touch with us at meetings-api@vonage.com. 


### Types of Callbacks

- **Room Expired** (`room:expired`): A notification about a room becoming inactive, which means sessions can no longer be created for it
- **Session Started** (`session:started`): A notification about a newly started session
- **Session Ended** (`session:ended`): A notification about a session that has just ended 
- **Recording Started** (`recording:started`): A notification about a recording beginning within a session 
- **Recording Ended** (`recording:ended`): A notification about a recording being stopped within a session 
- **Recording Uploaded** (`recording:uploaded`): A notification that a recording is ready to be downloaded 
- **Participant Joined** (`participant-joined`): A notification about someone entering a session 
- **Participant Left** (`session:participant:left`): A notification about someone leaving a session 

### Example Payloads 

#### Session Started
```
    "event": "session:started",
    "session_id": "2_MX40NjMzOTg5Mn5-MTYzNTg2ODQwODY4NH41cXIzMDdSa1BZa05BUDFpYnhxcTV4MCt-fg",
    "room_id": "b307d837-c0ce-4619-8c5c-70e418ef9693",
    "started_at": "2021-11-02T15:53:28.753Z"
```
#### Participant Joined
```
    "event": "session:participant:joined",
    "participant_id": "b424e1c4-e988-4ce2-8ab9-e3efea7de542",
    "session_id": "2_MX40NjMzOTg5Mn5-MTYzNTg2ODQwODY4NH41cXIzMDdSa1BZa05BUDFpYnhxcTV4MCt-fg",
    "room_id": "b307d837-c0ce-4619-8c5c-70e418ef9693",
    "name": "Avital",
    "type": "Guest",
    "is_host": true
```

#### Recording Started
```
    "event": "recording:started",
    "recording_id": "17461b93-f793-48a0-9392-7d82de40432f",
    "session_id": "2_MX40NjMzOTg5Mn5-MTYzNTg2ODUxNzIzNH5mOUVub3hPNCt6czlwQzdvaTYvbm5lOTN-fg"
```

#### Recording Uploaded
```
    "event": "recording:uploaded",
    "recording_id": "17461b93-f793-48a0-9392-7d82de40432f",
    "session_id": "2_MX40NjMzOTg5Mn5-MTYzNTg2ODUxNzIzNH5mOUVub3hPNCt6czlwQzdvaTYvbm5lOTN-fg",
    "started_at": "2021-11-02T15:55:35.000Z",
    "ended_at": "2021-11-02T15:56:13.000Z",
    "duration": 38,
    "url": "https://prod-meetings-recordings.s3.amazonaws.com/46339892/17461b93-f793-48a0-9392-7d82de40432f/archive.mp4?..."
```

## Recordings 

Recordings are associated with the session in which they occurred. To retrieve or manage recordings, you'll need the session ID, which can be found in the callbacks. 

### Retrieve All Recordings From A Session 

To get all recordings for the Session ID above, use the `sessions` endpoint: 

```
curl --location --request GET 'https://api-eu.vonage.com/beta/meetings/sessions/2_MX40NjMzOTg5Mn5-MTYzNTg2ODUxNzIzNH5mOUVub3hPNCt6czlwQzdvaTYvbm5lOTN-fg/recordings' \
--header 'Authorization: Basic YWFhMDEyOmFiYzEyMzQ1Njc4OQ==' \
--header 'Content-Type: application/json'
```

### Retrieve Individual Recording 

Once you have the recording ID, you can use the ``recordings`` endpoint: 

```
curl --location --request GET 'https://api-eu.vonage.com/beta/meetings/recordings/17461b93-f793-48a0-9392-7d82de40432f' \
--header 'Authorization: Basic YWFhMDEyOmFiYzEyMzQ1Njc4OQ==' \
--header 'Content-Type: application/json'
```    

### Delete A Recording 

Similarly, you can delete a recording with a DELETE action on the same endpoint. 

```
curl --location --request DELETE 'https://api-eu.vonage.com/beta/meetings/recordings/17461b93-f793-48a0-9392-7d82de40432f' \
--header 'Authorization: Basic YWFhMDEyOmFiYzEyMzQ1Njc4OQ==' \
--header 'Content-Type: application/json'
```    



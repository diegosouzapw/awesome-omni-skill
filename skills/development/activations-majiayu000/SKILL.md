---
name: activations
description: Query activation logs to check for errors and view volume
---

# Activations

Treasue Data's real time services include digital marketing activations. These are sent to various destinations. This skill can be used to query the database table that stores activations logs for both successful and failed activations, which is a source of useful information for the digital marketer.

# Description

## Requirements

In order to query the activation log we must know the parent segment the customer is interested in. A customer may have a number of parent segments so we must ask them to provide the one they are interested before making a query. A segment ID will be a numeric value like 411671.

The user must also have a correctly configured Treasure Data mcp server to enable the database lookup @treasuredata/mcp-server

In addition the api key with appropriate access to the database table should be available and configured.

## Database

The database name contains the parent segment ID and has this format cdp_audience_394649_rt. This is where you can plug in the parent segment the user gives in the request.

## Activations table

The activation log table is always called activations and is in the Parent Segment real time database.

## Schema

The activations table has the following schema. The format is of the schema below is:

1. Schema column index (example 1)
2. Column name (example time)
3. Query Column name (example time) (generally can be ignored for activations)
4. Data type (string, time, int)

1
time
time
int
2
delivered
delivered
string
--
3
status
status
long
--
4
timestamp
timestamp
long
--
5
activation_type
activation_type
string
--
6
log_time
log_time
long
--
7
journey_name
journey_name
string
--
8
journey_stage_name
journey_stage_name
string
--
9
activation_name
activation_name
string
--
10
rid
rid
string
--
11
error
error
string
--
12
activation_id
activation_id
string
--
13
event_id
event_id
string
--
14
response
response
string
--

For the purposes of making queries and talking about activations with the user here are the meanings of the columns.

time - epoch time of the activation send attempt
delivered - true if successfully sent and false otherwise
status - numeric status code corresponding to http response return codes
timestamp 
activation_type - the text name of the activation. td_webhook_out for example
log_time - 
journey_name - The real time journey this is related to. For example journey_11738
journey_stage_name - The stage of the real time journey. For example journey_stage_15055.
activation_name - Customer's name for the activation. For example first activation_14136
error - When an error occurs the text will be here. For example "AxiosError: Request failed with status code 404"
activation_id - Unique identifier of the activation.
event_id - Unique identifier for the event which triggered the activation.
response - Contains text that was returned from the activation server. This occurs whether or not the request succeeded. Example `{"success":false,"error":{"message":"Token \"c58dd43b-5bbc-4d21-a81a-e8c5643bcc18\" not found","id":""}}` It can be useful to show the response when errors are involved.

## Query tips

If the user does not specify a time range assume the last 24 hours. Use a where clause like `TD_INTERVAL(time, '-1d/now')`
When users are asking "What kind of errors are occuring" focus on unique error status and error texts so they can quickly identify issues.
Provide summary data where appropriate. For example the customer may ask for "Which journeys are triggering the most activations", or "group the number of activations by journey". Use unique and other SQL constructs to make useful aggregations on the data.







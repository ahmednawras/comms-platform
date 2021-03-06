---
layout: docs
title: "Channels"
---

# Channels

Comms can be issued over the following channels:

* Email
* SMS
* Print

Each time a comm is [triggered](events.html) the channel the comm is sent over is determined by the following factors:

## Channels in the Template

When a comm [template](templates.html) is published, the channels the comm is available over are specified. 

A template obviously must have at least one channel, which can be either SMS, email or print. 

## Customer Communication Preferences

Within My OVO customers set which channels they wish to receive comms over, they are able to do this for Service and Marketing comms separately.
 
For Service comms the customer is forced to choose either Print or Email, with SMS available as an additional option.

Unless specified by the customer in their communication preferences, comms will not be issued via said channel.

<div class="alert alert-info">
If a customer has no Service Communication Preferences this indicates that they have not actually set any, as either email or post must be selected, and therefore the customer's Communication Preferences are not taken into account.
</div>

[comment]: <> (This statement has to be reviewed.)
While the platform is only supporting notification comms, and only supporting email and SMS channels, if a customer has set their Service Communication Preferences to post rather than email, then post is removed from their list of Service Communication Preferences used by the platform. 

## Customer Profile

The profiles service provides us with contact details for a customer. 

In order to send an email or SMS, an email address or mobile number are required respectively, if these are not present then the comm can not be issued over that channel.

## Preferred Channels

When you [trigger](events.html) a comm you can provide a list of preferred channels by setting the `preferredChannels` field.

Note that this is only one of the inputs to the channel selection logic, so the comm may be issued over a channel not specified in this list.

### Example Triggered Event

The following example is triggering a MeterReads comm, specifying that we would like to send the comm over SMS if possible, followed by email and then any other channel.

```tut:silent
import com.ovoenergy.comms.model._
import com.ovoenergy.comms.triggered.Template
                                                                 
@Template("service", "example-for-docs", "0.1")
object MeterReadsComm
 
val data = MeterReadsComm(
  gas = None, // not a gas customer
  electricity = Some(MeterReadsComm.Electricity(
    meters = List(
      // customer has 2 electricity meters
      MeterReadsComm.Electricity.Meter(firstNumber = "123", secondNumber = "456"),
      MeterReadsComm.Electricity.Meter(firstNumber = "789", secondNumber = "123")
    )
  ))
)

val metadata = MetadataV2(
	createdAt = java.time.Instant.now(),
	eventId = java.util.UUID.randomUUID().toString,
	deliverTo = Customer("my-customer"),
	traceToken = java.util.UUID.randomUUID().toString,
	commManifest = MeterReadsComm.commManifest, // use the generated comm manifest in the companion object
	friendlyDescription = "awesome comm",
	source = "my amazing service",
	triggerSource = "my amazing service",
	canary = true,
	sourceMetadata = None
)

val preferredChannels = Some(List(SMS, Email))
val templateData: Map[String, TemplateData] = data.convertToTemplateData

val event = TriggeredV3(
  metadata = metadata, 
  templateData = templateData, 
  deliverAt = None, 
  expireAt = None, 
  preferredChannels = preferredChannels
)
```

## Cost

All things being equal the comm will be issued using the cheapest channel.

## The decision flow

The following diagram shows how the above are used to determine the channel to send the comm over.

![Channel selection logic flow](../img/comms-channels-logic.png)

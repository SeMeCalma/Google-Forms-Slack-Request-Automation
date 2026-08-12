# Workflow Documentation

## Google Forms → Zapier → Slack Automation

This document describes the internal workflow used in the **Google Forms → Slack Request Automation** project.

## Architecture

The system uses Google Forms as the input interface, Google Sheets as persistent response storage, Zapier as the automation platform, and Slack as the notification destination.

```text
                         ┌─────────────────┐
                         │  Google Sheets  │
                         │                 │
                         │ Response history│
                         └────────▲────────┘
                                  │
                                  │
┌──────────┐     ┌───────────────┴─┐
│   User   │────▶│   Google Forms  │
└──────────┘     └────────┬────────┘
                          │
                          │ New Form Response
                          ▼
                 ┌─────────────────┐
                 │     Zapier      │
                 │                 │
                 │ Trigger         │
                 │ Field Mapping   │
                 └────────┬────────┘
                          │
                          │ Send Channel Message
                          ▼
                 ┌─────────────────┐
                 │      Slack      │
                 │                 │
                 │ Team            │
                 │ Notification    │
                 └─────────────────┘
```

## Step 1 — User Submission

A user submits a request through Google Forms.

The form contains the following fields:

| Field   | Type       | Purpose                      |
| ------- | ---------- | ---------------------------- |
| Name    | Short text | Identifies the requester     |
| Email   | Short text | Provides contact information |
| Message | Long text  | Contains the request         |

## Step 2 — Response Storage

Google Forms records submitted responses.

The associated Google Sheets spreadsheet maintains a structured history containing information such as:

```text
Timestamp
Name
Email
Message
```

This separates persistent data storage from the notification system.

## Step 3 — Zapier Trigger

Zapier uses Google Forms as the workflow trigger.

Configuration:

```text
Application: Google Forms
Trigger Event: New Form Response
```

When Zapier detects a new response, it retrieves the data associated with that submission.

Example input:

```text
Name:
Camila Torres Ruiz

Email:
camila.torres@example.com

Message:
Hola, me gustaría consultar sobre el estado de mi solicitud pendiente.
```

The example uses synthetic test information.

## Step 4 — Data Mapping

Zapier maps values returned by Google Forms into the Slack action.

The mapping follows this structure:

```text
Google Forms                  Slack

Name            ────────────▶ Name
Email           ────────────▶ Email
Message         ────────────▶ Message
```

Because the fields are dynamically mapped, the automation does not depend on fixed names, email addresses, or messages.

Every new submission provides its own values.

## Step 5 — Slack Action

The second Zapier step uses:

```text
Application: Slack
Action Event: Send Channel Message
```

The notification template is:

```text
📩 Nueva solicitud recibida

👤 Nombre: {{Name}}
📧 Correo: {{Email}}

💬 Mensaje: {{Message}}
```

Zapier replaces the mapped variables with the values from the current Google Forms response.

For example:

```text
📩 Nueva solicitud recibida

👤 Nombre: Camila Torres Ruiz
📧 Correo: camila.torres@example.com

💬 Mensaje: Hola, me gustaría consultar sobre el estado de mi solicitud pendiente.
```

## Step 6 — Delivery

Zapier sends the generated message to the configured Slack channel.

The team can immediately see:

* Who submitted the request.
* Their contact email.
* The content of their request.

The original response remains available through the Google Forms response history and associated Google Sheets spreadsheet.

## Workflow Summary

```text
FORM SUBMISSION
      │
      ▼
NEW GOOGLE FORM RESPONSE
      │
      ├──────────────▶ GOOGLE SHEETS
      │                 Store response
      │
      ▼
ZAPIER TRIGGER
      │
      ▼
EXTRACT FORM DATA
      │
      ├── Name
      ├── Email
      └── Message
      │
      ▼
MAP DATA TO SLACK TEMPLATE
      │
      ▼
SEND CHANNEL MESSAGE
      │
      ▼
SLACK NOTIFICATION
```

## Current Limitations

Version 1.0 intentionally keeps the workflow simple.

Currently, the automation does not:

* Categorize requests.
* Determine priority.
* Route requests to different channels.
* Send automatic responses to users.
* Use AI to analyze messages.
* Integrate with a CRM.
* Implement advanced error handling.

These features could be added in future iterations.

## Test Result

The workflow was tested with multiple synthetic submissions.

**Result: Successful**

Google Forms responses were correctly detected and the corresponding dynamic notifications were successfully delivered to Slack.

# 1. **Structure of an FCM Notification**

• **Notification Payload**: Contains title and body fields for UI display.
• **Data Payload**: Contains custom key-value pairs for flexible handling within the app.
# **2. Delivery Scenarios and Behaviour**

**a) App Not Launched (Cold State)**

• FCM delivers the message to the Android system, which displays the notification automatically based on the notification payload.

• If a **data-only payload** is used, no notification is displayed automatically; it is silently delivered and requires app intervention when launched.

**b) App in Background**

• **Notification Payload**: FCM automatically displays the notification in the system tray.

• **Data Payload (or combined payload)**: Both notification and data payloads are delivered. The notification part is handled by the system (shown automatically), and the data part is delivered to the app when it is opened.

**c) App in Foreground**

• **Notification Payload**: No system tray notification is automatically displayed. Instead, the message is delivered to the app via onMessageReceived() in FirebaseMessagingService, and you need to manually show the notification.

• **Data Payload**: Delivered directly to onMessageReceived() for you to handle as needed (e.g., show a notification or perform an in-app action).
# **3. Consistent Display of Notifications**

To ensure a consistent user experience (e.g., always showing the sender’s name when a message is received):

• **Use a data payload only**: This gives you full control over how the notification appears in all states (foreground, background, and when the app is not launched).

• Implement a FirebaseMessagingService to handle the data payload, construct the notification using Android’s Notification APIs, and show it based on your desired format, including sender details and any additional data.
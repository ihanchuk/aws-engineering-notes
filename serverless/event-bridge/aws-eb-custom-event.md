```JS
import { EventBridgeClient, PutEventsCommand } from "@aws-sdk/client-eventbridge";

// Создаём клиента
const client = new EventBridgeClient({ region: "us-east-1" });

async function fireCustomEvent() {
  const params = {
    Entries: [
      {
        EventBusName: "Shop:Events", 
        Source: "App:Mobile",  
        DetailType: "Order:Confirmed",
        Time: new Date(),
        Detail: JSON.stringify({
          userId,
          action: "confirmed_order",
          orderId: "ORD-2323-3478-4567"
        }),
      },
    ],
  };

  try {
    const command = new PutEventsCommand(params);
    const response = await client.send(command);
    console.log("Event sent:", response);
  } catch (err) {
    console.error("Error sending event:", err);
  }
}

fireCustomEvent();

```

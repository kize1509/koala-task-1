# CONCEPT
- Encapsulating all required event listeners into an entity.  
- Extractign the event data.  
- Creating an envelope onbject to hold all event data.  

**Envelope Object Suggested Structure**

```json
{
    "timestamp": "2023-10-01T12:00:00Z",
    "event_type": "event_type",
    "context" : {
        "component": "component_name",
        "file_path": "path/to/file",
        "user_id": "user_id",
    },
    "event_id": "event_id",
    "payload":  {
        //event specific data
    }
}
```
- The payload object contains event-specific details.
- The context object holds additional event context.
- With this approach, we have an envelope object that is generic and can be used for all events.


## Storing and Sending data
- The idea is to design a system that securely stores events.  
- Since events can occur in rapid succession, it is necessary to store them locally and send them in batches.  
- While the system is running, data stored in memory is volatile, so it must be persisted to disk.  

### Storage Considerations

**Since the data is not structured in a completely predictable way, it needs to be stored in a manner that does not depend on a fixed schema:**

1. We can use a NoSQL database on the server to store the data persistently. For temporary local storage, a simple file system may suffice.  
2. By employing a Write-Ahead Log (WAL) approach, you can store the data in a file and retrieve it if needed. Additionally, consider encrypting the logs or using secure file permissions to enhance security.  

### Batching Requests

- Design a Memtable-like structure to act as a buffer for data before it is sent to the server.

- Upon receiving an event, perform a WAL operation to store the data in a file and then add the data to the buffer.

- When the buffer reaches a configured size, send the data to the server, then clear the buffer and delete the logs.

- Data transmission should be handled in a separate thread to ensure the main thread remains unblocked and continues receiving events.

- WAL is our backup plan. If the data is not sent to the server, we can retrieve it from the WAL file.

- When data is successfully sent to the server, delete the corresponding WAL entry.

- WAL deletion and creation needs to be done in a separate thread to avoid blocking the main thread.

## Data storing 

- Each data object has an unexpected object payload. 
- So the best way to store the data is to use a NoSQL database on the server.

### Additional Points

- Structured data part allows us to cluster the data and use it for analytics.
- The unstructured data part allows us to store the data in a flexible way and perform fast write operations.

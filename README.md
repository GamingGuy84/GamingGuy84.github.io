# Notwork

Notwork is a lightweight Roblox networking library that adds type safety, schema validation, and compact binary serialization to RemoteEvents, without changing how you use them.

---

## Features

- Strongly typed remote events
- Schema-based data validation
- Automatic serialization via buffers
- Smaller payloads compared to raw tables
- Shared definitions between client and server
- Simple event-driven API
- Support for Roblox types (Vector3, CFrame, etc.)

---

## Installation

Place `Notwork` in a shared location (e.g. `ReplicatedStorage`) and require it on both client and server.

Shared Packet Definition
```lua
local Notwork = require(path.to.Notwork)

local Packets = {}

--Define a packet once and use it on both client and server.
Packets.Packet = Notwork.DefinePacket({
    Foo = Notwork.string,
    Foo2 = Notwork.vector3
})

return Packets
```

Server Example
```lua
local Packets = require(path.to.SharedPackets)

local Packet = Packets.Packet

Packet.OnClientEvent:Connect(function(player, data)
    print("Received from client:", data)

    Packet:FireAllClients({
        Foo = "Hello from the Server!",
        Foo2 = Vector3.new(0, -5, 25)
    })
end)
```

Client Example
```lua
local Packets = require(path.to.SharedPackets)

local Packet = Packets.Packet

Packet.OnServerEvent:Connect(function(data)
    print("Received from server:", data)
end)

Packet:FireServer({
    Foo = "Hello from the Client!",
    Foo2 = Vector3.new(0, 5, 0)
})
```

---

## API

### Packet Constructors
```lua
local Notwork = require(path.to.Notwork)

-- Reliable (guaranteed delivery)
Notwork.DefinePacket(schema: {[string]: Notwork.DataType}) -> Packet

-- Unreliable (may drop, lower overhead)
Notwork.DefineUnreliablePacket(schema: {[string]: Notwork.DataType}) -> Packet

--Example:
local ReliablePacket = Notwork.DefinePacket({
    Foo = Notwork.string,
    Foo2 = Notwork.vector3
})

local UnreliablePacket = Notwork.DefineUnreliablePacket({
    Foo = Notwork.string,
    Foo2 = Notwork.vector3
})
```

### Packet Methods
```lua
local Packet = require(path.to.SharedPackets).Packet

-- Data has type annotations with intellisense, allowing you to see data types based on the packet.

-- Can only be called from the client
Packet:FireServer(Data: {[string]: any})

-- Can only be called from the server
Packet:FireClient(Player: Player, Data : {[string]: any})
Packet:FireAllClients(Data : {[string]: any})
```

### Packet Events

```lua
local Packet = require(path.to.SharedPackets).Packet

-- Can only be listened to on the server
Packet.OnServerEvent:Connect(function(Player: Player, Data: {[string]: any})

-- Can only be listened to on the client
Packet.OnClientEvent:Connect(function(Data: {[string]: any})

```
---

## Reliable vs Unreliable

Similar to Roblox's RemoteEvents, reliable and unreliable events both have the purpose of networking, but do it in slightly different ways.

Reliable (`DefinePacket`)
- Guaranteed delivery
- Used for gameplay-critical data

Unreliable (`DefineUnreliablePacket`)
- May drop packets
- Lower latency / overhead
- Used for frequent updates (e.g positions, effects)

---

## How It Works

Notwork serializes structured Lua tables into compact binary buffers before sending them over Roblox RemoteEvents.

This provides:

- Reduced network payload size

- Faster replication of structured data

- Strict schema validation

- Consistent type handling between client and server

Instead of sending raw tables, data is encoded into a compact buffer format and decoded on the receiving side using the shared schema.

---

## Why Not Just Use RemoteEvents?

RemoteEvents are:
- Untyped
- Unvalidated
- Inefficient for structured data

This leads to:
- Silent bugs
- Inconsistent data formats
- Larger network costs

Notwork solves all three without changing your workflow.

---

## Supported Types
`Notwork.string`

`Notwork.number`

`Notwork.any`

`Notwork.bool`

`Notwork.f32`

`Notwork.f64`

`Notwork.i8`

`Notwork.i16`

`Notwork.i32`

`Notwork.u16`

`Notwork.u32`

`Notwork.instance`

`Notwork.vector2`

`Notwork.vector2int16`

`Notwork.vector3`

`Notwork.vector3int16`

`Notwork.udim`

`Notwork.udim2`

`Notwork.cframe`

---

## Limitations

- Packets must be defined ahead of time
  
- Not intended for dynamic runtime packet creation

- Does NOT have any Disconnect options for Packets

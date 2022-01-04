# Configuration for mediator agent (Node.js), edge agents (React Native), and ledger (VON) using Aries Framework JavaScript.

## Goals
1. Run a Node.js agent that can mediate connections and also issue verifiable credentials to other agents.
2. Run two React Native agents that both use the Node.js agent as a mediator.
3. Create a mediated connection between the two React Native agents and send basic messages between them.
4. Issue verifiable credentials from the Node.js agent to the React Native agents.
5. Allow the React Native agents to request, present, and verify proof of each other's credentials.

## Development environment
- OS: Ubuntu 18.04
- Path to project: `/server`
- Local IP address: `192.168.1.72`

## Initial Setup
### Prerequisites
1. Installed Libindy for Linux to be used by Node.js agent

### Node.js
1. Cloned https://github.com/hyperledger/aries-framework-javascript into `/server`
2. Renamed `/server/aries-framework-javascript` to `/server/afj`

### React Native
1. Created a new React Native project at `/server/mywallet`
2. Added the following packages to the project:
  - `@aries-framework/core`
  - `@aries-framework/react-native`
  - `react-native-fs`
  - `react-native-get-random-values`
  - `indy-sdk-react-native`
3. Installed Libindy for Android & iOS according to the guides

    iOS: https://github.com/hyperledger/aries-framework-javascript/blob/main/docs/libindy/ios.md
    
    Android: https://github.com/hyperledger/aries-framework-javascript/blob/main/docs/libindy/android.md

### VON
1. Cloned https://github.com/bcgov/von-network into `/server/von-network`
2. Built the von-network project
```
cd /server/von-network
./manage build
```
3. Started VON
```
cd /server/von-network
./manage start --logs
```

4. The VON webserver is now accessible at either of these URLs:
- `http://localhost:9000/`
- `http://192.168.1.72:9000/`

## Development
### Mediator agent configuration on Node.js
1. Created `/server/afj/config/von/pool-config.ts`
```typescript
"use strict";
exports.__esModule = true;
var genesis_file_1 = require("./genesis-file");
var config = {
    id: 'VON',
    genesisTransactions: genesis_file_1["default"],
    isProduction: false
};
exports["default"] = config;
```
2. Visited the VON webserver UI located at `http://localhost:9000/` and clicked the "Genesis Transaction" button to download the JSON needed for the genesis file.
3. Created `/server/afj/config/von/genesis-file.ts` using the downloaded genesis transaction JSON.
```typescript
export default `{"reqSignature":{},"txn":{"data":{"data":{"alias":"Node1","blskey":"4N8aUNHSgjQVgkpm8nhNEfDf6txHznoYREg9kirmJrkivgL4oSEimFF6nsQ6M41QvhM2Z33nves5vfSn9n1UwNFJBYtWVnHYMATn76vLuL3zU88KyeAYcHfsih3He6UHcXDxcaecHVz6jhCYz1P2UZn2bDVruL5wXpehgBfBaLKm3Ba","blskey_pop":"RahHYiCvoNCtPTrVtP7nMC5eTYrsUA8WjXbdhNc8debh1agE9bGiJxWBXYNFbnJXoXhWFMvyqhqhRoq737YQemH5ik9oL7R4NTTCz2LEZhkgLJzB3QRQqJyBNyv7acbdHrAT8nQ9UkLbaVL9NBpnWXBTw4LEMePaSHEw66RzPNdAX1","client_ip":"172.17.0.1","client_port":9702,"node_ip":"172.17.0.1","node_port":9701,"services":["VALIDATOR"]},"dest":"Gw6pDLhcBcoQesN72qfotTgFa7cbuqZpkX3Xo6pLhPhv"},"metadata":{"from":"Th7MpTaRZVRYnPiabds81Y"},"type":"0"},"txnMetadata":{"seqNo":1,"txnId":"fea82e10e894419fe2bea7d96296a6d46f50f93f9eeda954ec461b2ed2950b62"},"ver":"1"}
{"reqSignature":{},"txn":{"data":{"data":{"alias":"Node2","blskey":"37rAPpXVoxzKhz7d9gkUe52XuXryuLXoM6P6LbWDB7LSbG62Lsb33sfG7zqS8TK1MXwuCHj1FKNzVpsnafmqLG1vXN88rt38mNFs9TENzm4QHdBzsvCuoBnPH7rpYYDo9DZNJePaDvRvqJKByCabubJz3XXKbEeshzpz4Ma5QYpJqjk","blskey_pop":"Qr658mWZ2YC8JXGXwMDQTzuZCWF7NK9EwxphGmcBvCh6ybUuLxbG65nsX4JvD4SPNtkJ2w9ug1yLTj6fgmuDg41TgECXjLCij3RMsV8CwewBVgVN67wsA45DFWvqvLtu4rjNnE9JbdFTc1Z4WCPA3Xan44K1HoHAq9EVeaRYs8zoF5","client_ip":"172.17.0.1","client_port":9704,"node_ip":"172.17.0.1","node_port":9703,"services":["VALIDATOR"]},"dest":"8ECVSk179mjsjKRLWiQtssMLgp6EPhWXtaYyStWPSGAb"},"metadata":{"from":"EbP4aYNeTHL6q385GuVpRV"},"type":"0"},"txnMetadata":{"seqNo":2,"txnId":"1ac8aece2a18ced660fef8694b61aac3af08ba875ce3026a160acbc3a3af35fc"},"ver":"1"}
{"reqSignature":{},"txn":{"data":{"data":{"alias":"Node3","blskey":"3WFpdbg7C5cnLYZwFZevJqhubkFALBfCBBok15GdrKMUhUjGsk3jV6QKj6MZgEubF7oqCafxNdkm7eswgA4sdKTRc82tLGzZBd6vNqU8dupzup6uYUf32KTHTPQbuUM8Yk4QFXjEf2Usu2TJcNkdgpyeUSX42u5LqdDDpNSWUK5deC5","blskey_pop":"QwDeb2CkNSx6r8QC8vGQK3GRv7Yndn84TGNijX8YXHPiagXajyfTjoR87rXUu4G4QLk2cF8NNyqWiYMus1623dELWwx57rLCFqGh7N4ZRbGDRP4fnVcaKg1BcUxQ866Ven4gw8y4N56S5HzxXNBZtLYmhGHvDtk6PFkFwCvxYrNYjh","client_ip":"172.17.0.1","client_port":9706,"node_ip":"172.17.0.1","node_port":9705,"services":["VALIDATOR"]},"dest":"DKVxG2fXXTU8yT5N7hGEbXB3dfdAnYv1JczDUHpmDxya"},"metadata":{"from":"4cU41vWW82ArfxJxHkzXPG"},"type":"0"},"txnMetadata":{"seqNo":3,"txnId":"7e9f355dffa78ed24668f0e0e369fd8c224076571c51e2ea8be5f26479edebe4"},"ver":"1"}
{"reqSignature":{},"txn":{"data":{"data":{"alias":"Node4","blskey":"2zN3bHM1m4rLz54MJHYSwvqzPchYp8jkHswveCLAEJVcX6Mm1wHQD1SkPYMzUDTZvWvhuE6VNAkK3KxVeEmsanSmvjVkReDeBEMxeDaayjcZjFGPydyey1qxBHmTvAnBKoPydvuTAqx5f7YNNRAdeLmUi99gERUU7TD8KfAa6MpQ9bw","blskey_pop":"RPLagxaR5xdimFzwmzYnz4ZhWtYQEj8iR5ZU53T2gitPCyCHQneUn2Huc4oeLd2B2HzkGnjAff4hWTJT6C7qHYB1Mv2wU5iHHGFWkhnTX9WsEAbunJCV2qcaXScKj4tTfvdDKfLiVuU2av6hbsMztirRze7LvYBkRHV3tGwyCptsrP","client_ip":"172.17.0.1","client_port":9708,"node_ip":"172.17.0.1","node_port":9707,"services":["VALIDATOR"]},"dest":"4PS3EDQ3dW1tci1Bp6543CfuuebjFrg36kLAUcskGfaA"},"metadata":{"from":"TWwCRQRZ2ZHMJFn9TzLp7W"},"type":"0"},"txnMetadata":{"seqNo":4,"txnId":"aa5e817d7cc626170eca175822029339a444eb0ee8f0bd20d3b0b76e566fb008"},"ver":"1"}`
```
4. Create `/server/afj/mediator.ts` based on [/samples/mediator.ts](https://github.com/hyperledger/aries-framework-javascript/blob/main/samples/mediator.ts) from the aries-framework-javascript repository
```typescript
import { InitConfig } from "@aries-framework/core";

import * as express from "express";
import { Server } from "ws";

import { TestLogger } from "./packages/core/tests/logger";

import {
  HttpOutboundTransport,
  Agent,
  LogLevel,
  AgentConfig,
  WsOutboundTransport,
  AutoAcceptProof,
  AutoAcceptCredential
} from "@aries-framework/core";
import {
  HttpInboundTransport,
  agentDependencies,
  WsInboundTransport,
} from "@aries-framework/node";

import indyConfig from "./configs/von/pool-config";

const port = 3001;

const app = express();
const socketServer = new Server({ noServer: true });

const endpoints = [`http://192.168.1.72:${port}`, `ws://192.168.1.72:${port}`];

const logger = new TestLogger(LogLevel.info);

const agentConfig: InitConfig = {
  endpoints,
  label: process.env.AGENT_LABEL || "Private Data Vault",
  walletConfig: {
    id: process.env.WALLET_NAME || "PrivateDataVault",
    key: process.env.WALLET_KEY || "PrivateDataVault",
  },
  autoAcceptProofs: AutoAcceptProof.ContentApproved,
  autoAcceptConnections: true,
  autoAcceptMediationRequests: true,
  indyLedgers: [indyConfig],
  genesisTransactions: indyConfig.genesisTransactions,
  publicDidSeed: "privatedatavaultpublicdidseeddid",
  autoAcceptCredentials: AutoAcceptCredential.ContentApproved,
  logger,
};

// Set up agent
const agent = new Agent(agentConfig, agentDependencies);
const config = agent.injectionContainer.resolve(AgentConfig);

// Create transports
const httpInboundTransport = new HttpInboundTransport({ app, port });
const httpOutboundTransport = new HttpOutboundTransport();
const wsInboundTransport = new WsInboundTransport({ server: socketServer });
const wsOutboundTransport = new WsOutboundTransport();

// Register transports
agent.registerInboundTransport(httpInboundTransport);
agent.registerOutboundTransport(httpOutboundTransport);
agent.registerInboundTransport(wsInboundTransport);
agent.registerOutboundTransport(wsOutboundTransport);

const runAgent = async () => {
  await agent.initialize();

  // forward upgrade on http server to ws server
  httpInboundTransport.server?.on("upgrade", (request, socket, head) => {
    socketServer.handleUpgrade(request, socket, head, (socket) => {
      socketServer.emit("connection", socket, request);
    });
  });
};

httpInboundTransport.app.get('/invitation', async (req, res) => {
  if (typeof req.query.c_i === 'string') {
    const invitation = await ConnectionInvitationMessage.fromUrl(req.url)
    res.send(invitation.toJSON())
  } else {
    const { invitation } = await agent.connections.createConnection()

    const httpEndpoint = config.endpoints.find((e) => e.startsWith('http'))

    var inv = invitation.toUrl({ domain: httpEndpoint + '/invitation' })

    res.send(inv)
  }
})

runAgent();
```
5. Build and run the mediator.ts file on Node.js with `tsc mediator.ts ; node mediator.js`
6. Test that the agent is working
- Visit `http://192.168.1.72:3001/invitation`
- Receive invitation URL `http://192.168.1.72:3001/invitation?c_i=eyJAdHlwZSI6Imh0dHBzOi8vZGlkY29tbS5vcmcvY29ubmVjdGlvbnMvMS4wL2ludml0YXRpb24iLCJAaWQiOiIyOGMxZTBmNS01YzVhLTRkYjMtYjhkZC02Y2IwZDgxN2Y4MTIiLCJsYWJlbCI6Ik1pZ3JhdGUgRGF0YSBWYXVsdCIsInJlY2lwaWVudEtleXMiOlsiQmNhQ3E2dzdiZmtvZGI4aUxhR0tTQXMzaEZEOEpENHVVZlNDeXd1REdUcTEiXSwic2VydmljZUVuZHBvaW50Ijoid3M6Ly8xOTIuMTY4LjEuNzI6MzAwMSIsInJvdXRpbmdLZXlzIjpbXX0`
- Visit the invitation URL to view the invitation contents
```json
{
  "@type": "https://didcomm.org/connections/1.0/invitation",
  "@id": "28c1e0f5-5c5a-4db3-b8dd-6cb0d817f812",
  "label": "Private Data Vault",
  "recipientKeys": [
    "BcaCq6w7bfkodb8iLaGKSAs3hFD8JD4uUfSCywuDGTq1"
  ],
  "serviceEndpoint": "ws://192.168.1.72:3001",
  "routingKeys": []
}
```

The mediator agent is now running on Node.js and able to create connection invitations.

### Configuration of edge agent on React Native
1. Copied the `/server/afj/configs` folder that contains the VON genesis and pool files to the React Native project at `/server/mywallet/configs`
2. Modified the automatically created `/server/mywallet/App.js` file and added the ability to initialize an agent with a customized wallet label
```javascript
 import {
  Agent,
  HttpOutboundTransport,
  MediatorPickupStrategy,
  WsOutboundTransport,
  AutoAcceptProof,
  AutoAcceptCredential,
  ConsoleLogger,
} from "@aries-framework/core";
import { agentDependencies } from "@aries-framework/react-native";

import React from "react";
import type { Node } from "react";
import { SafeAreaView, StatusBar, Text, View, Button } from "react-native";

import indyConfig from "./configs/von/pool-config";

const App: () => Node = () => {
  const [agent, setAgent] = React.useState(null);
  const [myConnections, setMyConnections] = React.useState([]);

  const initializeAgent = async (walletLabel) => {
    var invite = await fetch("http://192.168.1.72:3001/request");

    var data = await invite.text();

    const newAgent = new Agent(
      {
        label: "My Wallet " + walletLabel,
        mediatorConnectionsInvite: data,
        mediatorPickupStrategy: MediatorPickupStrategy.Implicit,
        walletConfig: { id: "mywalletid", key: "mywalletkey" },
        autoAcceptConnections: true,
        autoAcceptProofs: AutoAcceptProof.ContentApproved,
        autoAcceptCredentials: AutoAcceptCredential.ContentApproved,
        logger: new ConsoleLogger(LogLevel.trace),
        indyLedgers: [indyConfig],
        genesisTransactions: indyConfig.genesisTransactions,
      },
      agentDependencies
    );

    const wsTransport = new WsOutboundTransport();
    const httpTransport = new HttpOutboundTransport();

    newAgent.registerOutboundTransport(wsTransport);
    newAgent.registerOutboundTransport(httpTransport);

    await newAgent.initialize();

    setAgent(newAgent);
  };

  return (
    <SafeAreaView>
      <StatusBar barStyle={"light-content"} />
      <View>
        <Text>My Wallet</Text>
      </View>
      <View>
        <Button
          onPress={() => {
            initializeAgent("Alice");
          }}
          title="Initialize Agent with name Alice"
        />
        <Button
          onPress={() => {
            initializeAgent("Bob");
          }}
          title="Initialize Agent with name Bob"
        />
      </View>
    </SafeAreaView>
  );
};

export default App;
```
3. Launched an Android emulator using `sudo ./emulator -avd Pixel_3a_API_30_x86` to create a *Pixel 3a* device
4. Installed the React Native app on the emulator using `npx react-native run-android`
5. Launched a second emulator using `sudo ./emulator -avd Pixel_3a_API_30_x86 -read-only` to create a second *Pixel 3a* device (since the same command was used to launch the emulator, the React Native app is already installed on the second emulator when it launches)
6. Opened the React Native app on both emulators
7. Both devices display the "My Wallet" application interface which consists of two buttons "Initialize Agent with name Alice" and "Initialize Agent with name Bob"
8. Tapped "Initialize Agent with name Alice" on the first device and tapped "Initialize Agent with name Bob" on the second device

The React Native "My Wallet" app is now running on both emulated devices.

The first device initialized the agent using the wallet label *My Wallet Alice* and the second device initialized the agent using the wallet label *My Wallet Bob*

The *My Wallet Alice* agent will be now be referred to as *Alice's agent* and the *My Wallet Bob* agent will now be referred to as *Bob's agent*.

### Confirming both "My Wallet" React Native app connections to the Node.js mediator
1. Added a method to the `/server/afj/mediator.ts` file to list all connections to the mediator
```typescript
httpInboundTransport.app.get("/connections", async (req, res) => {
  var connections = await agent.connections.getAll();

  var conns = [];

  connections.forEach((c, i) => {
    conns.push({
      label: c.theirLabel,
      id: c.id,
    });
  });

  var list = "";

  conns.forEach((conn, i) => {
    list = list + conn.label + `<br />`;
  });

  res.send(list);
});
```
2. Visit http://192.168.1.72:3001/connections in a browswer to display a list of connections to the mediator
3. The browswer shows the following output
```
Private Wallet Alice
Private Wallet Bob
```
The "My Wallet" React Native app running on both emulators is successfully connected to the Node.js mediator

### Establish a mediated connection between the two "My Wallet" React Native apps
1. Added a method to the `/server/mywallet/App.js` file to create connection invitations from the React Native app
```javascript
const createInvite = async () => {
  const {invitation, connectionRecord} =
    await agent.connections.createConnection({
      autoAcceptConnection: true,
      alias: 'inviter',
    });

  var inv = invitation.toUrl({
    domain: 'http://192.168.1.72:3001' + '/invitation',
  });

  setAgentInvite(inv);
};
```
2. Added a variable to store a created invitation
```javascript
const [agentInvite, setAgentInvite] = React.useState('');
```
3. Added a button to call the `createInvite()` method
```javascript
<View>
  <Button
    onPress={() => {
      createInvite();
    }}
    title="Create Invite"
  />
</View>
```
4. Added a text input field that will serve two purposes:
- Inviter
  -  Display the value of the invitation URL stored in the `agentInvite` variable after pressing the "Create Invite" button
- Invitee
  -  Allow pasting of an invitation URL to store it in the `agentInvite` variable
```javascript
<View>
  <TextInput
    value={agentInvite}
    onChangeText={text => {
      setAgentInvite(text);
    }}
  />
</View>
```
5. Added a method to accept the connection invitation stored in the `agentInvite` variable
```javascript
const acceptInvite = async () => {
  agent.connections.receiveInvitationFromUrl(agentInvite, {
    autoAcceptConnection: true,
  });
};
```
6. Added a button to call the `acceptInvite()` method
```javascript
<View>
  <Button
    onPress={() => {
      acceptInvite();
    }}
    title="Accept Invite"
  />
```
7. Pressed the "Create Invite" button Alice's agent
8. The value of the newly created invitation URL is stored in the `agentInvite` variable on Alice's agent and appears in the text input field on Alice's agent
```
http://192.168.1.72:3001/invitation?c_i=eyJAdHlwZSI6Imh0dHBzOi8vZGlkY29tbS5vcmcvY29ubmVjdGlvbnMvMS4wL2ludml0YXRpb24iLCJAaWQiOiI1ODNhYzM0Zi0wN2NjLTQ2MWQtOTZhNC0zNGRmYTM2NDcxODYiLCJsYWJlbCI6Ik1pZ3JhdGUgV2FsbGV0IEFsaWNlIiwicmVjaXBpZW50S2V5cyI6WyI1VUFMOEpUaVZ2N3A4alBVUUFXVVFOR3lMTlc0SHVyMWtYREI1eTF4Vnl6aCJdLCJzZXJ2aWNlRW5kcG9pbnQiOiJodHRwOi8vMTkyLjE2OC4xLjcyOjMwMDEiLCJyb3V0aW5nS2V5cyI6WyJDY3dNWXdzMUtKUG5VQjlEMXpMWm80N2ZwQ2ExeW1pUHhXM21ETjVLVGJtWSJdfQ
```
9. Visited the newly created invitation URL in the browser to display its contents
```json
{
  "@type": "https://didcomm.org/connections/1.0/invitation",
  "@id": "583ac34f-07cc-461d-96a4-34dfa3647186",
  "label": "Private Wallet Alice",
  "recipientKeys": [
    "5UAL8JTiVv7p8jPUQAWUQNGyLNW4Hur1kXDB5y1xVyzh"
  ],
  "serviceEndpoint": "http://192.168.1.72:3001",
  "routingKeys": [
    "CcwMYws1KJPnUB9D1zLZo47fpCa1ymiPxW3mDN5KTbmY"
  ]
}
```
10. Pasted the newly created invitation URL into the text input field on Bob's agent
11. Pressed the "Accept Invite" button on Bob's agent

### Confirming the connection between the two React Native edge agents
1. Added a method to the `/server/mywallet/App.js` file to display a list of connections for the React Native app
```javascript
const listConnections = async () => {
  var connections = await agent.connections.getAll();

  var connectionList = [];

  connections.forEach((c, i) => {
    connectionList.push({
      name: c.theirLabel,
      id: c.id,
    });
  });

  setMyConnections(connectionList);
};
```
2. Added a variable to store the list of connections
```javascript
const [myConnections, setMyConnections] = React.useState([]);
```
3. Added a button to call the `getConnections()` method
```javascript
<View>
  <Button
    onPress={() => {
      listConnections();
    }}
    title="List Connections"
  />
</View>
```
4. Added a FlatList to display the list of connections
```javascript
<FlatList
    data={myConnections}
    renderItem={({item}) => (
    <View style={{flexDirection: 'row', marginBottom: 20}}>       
        <Text>{item.name}</Text>
    </View>
    )}
    keyExtractor={item => item.id}
/>
```
Pressing "List Connections" on Alice's agent shows the following:
```
Private Data Vault
Private Wallet Bob
```
Pressing "List Connections" on Bob's agent shows the following:
```
Private Data Vault
Private Wallet Alice
```
The two React Native edge agents have connected succesfully using the invitation that was created by Alice's agent and pasted into Bob's agent.

### Basic messaging configuration
1. Added a method to the `/server/mywallet/App.js` file to send a basic message to a specified connection ID
```javascript
const sendMessage = (conn,content) => {
  agent.basicMessages.sendMessage(conn, content);
};
```
2. Added a text input field to type new messages into
```javascript
<View>
  <TextInput
    value={message}
    onChangeText={text => {
      setMessage(text);
    }}
    placeholder="type message here"
  />
</View>
```
3. Added a variable to store the typed message
```javascript
const [message, setMessage] = React.useState('');
```
4. Added a "Send Message" button to each connection displayed in the FlatList to call the `sendMessage()` method with the ID of that connection along with the content of the typed message
```javascript
<FlatList
    data={myConnections}
    renderItem={({item}) => (
    <View style={{flexDirection: 'row', marginBottom: 20}}>    
        <Button
          onPress={() => {
            sendMessage(item.id,message);
          }}
          title="Send Message"
        />
        <Text>{item.name}</Text>
    </View>
    )}
    keyExtractor={item => item.id}
/>
```
5. Added `BasicMessageEventsTypes` to the list of imports
```javascript
import { BasicMessageEventTypes } from "@aries-framework/core";
```
6. Added a listener for when a basic message is received by the edge agent to display an alert with the content of the basic message
```javascript
agent.events.on(BasicMessageEventTypes.BasicMessageStateChanged, (e) => {
  if (e.payload.basicMessageRecord.role === "receiver") {
    alert(e.payload.message.content);
  }
});

```
7. Added a listener to the `/server/afj/mediator.ts` file to log basic messages received by the mediator to the console and respond to the sender with a basic message that says "message received"
```javascript
agent.events.on(BasicMessageEventTypes.BasicMessageStateChanged, (e) => {
  if (e.payload.basicMessageRecord.role === "receiver") {
    console.log(e.payload.message.content)
    agent.basicMessages.sendMessage(e.payload.basicMessageRecord.connectionId, 'message received')
  }
})
```

### Testing basic messaging between edge agents and the mediator
Alice
1. Typed a message saying "Hello from Alice" into the message text input field on Alice's agent
2. Pressed the "Send Message" button next to "Private Data Vault" in the list of connections on Alice's agent
3. The console for the mediator running on Node.js displays "Hello from Alice"
4. A return message saying "message received" is sent from the mediator to Alice's agent
5. Alice's agent displays an alert that says "message received"

Bob
1. Typed a message saying "Hello from Bob" into the message text input field on Bob's agent
2. Pressed the "Send Message" button next to "Private Data Vault" in the list of connections on Bob's agent
3. The console for the mediator running on Node.js displays "Hello from Bob"
4. A return message saying "message received" is sent from the mediator to Bob's agent
5. Bob's agent displays an alert that says "message received"

Both Alice and Bob can successfully exchange basic messages *with the mediator*.

### Testing basic messaging between the two edge agents
Alice to Bob
1. Typed a message saying "Hello Bob this is Alice" into the message text input field on Alice's agent
2. Pressed "Send Message" next to "Private Wallet Bob" in the list of connections on Alice's agent
3. Bob's agent displays an alert that says "Hello Bob this is Alice"

Bob to Alice
1. Typed a message saying "Hello Alice this is Bob" into the message text input field on Bob's agent
2. Pressed "Send Message" next to "Private Wallet Alice" in the list of connections on Bob's agent
3. Alice's agent displays an alert that says "Hello Alice this is Bob"

Both Alice's agent and Bob's agent can successfully exchange basic messages *with each other*.

### Registering a new schema and credential definition on the ledger
The Indy ledger running on the local VON network is permissioned to ony allow Trust Anchors or Trustees to write to it.

The DID and Verkey of the Node.js mediator agent must be registered / authenticated as an endorser via the VON webserver UI before the mediator will be allowed to write to the ledger.

1. Created a method in `/server/afj/mediator.ts' to retrieve the mediator's public DID and Verkey
```typescript
httpInboundTransport.app.get('/publicdid', async (req, res) => {  
  res.send(agent.publicDid)
})
```
2. Visited `http://localhost:3001/publicdid` in a browser to display the public DID and Verkey for the mediator
```json
{"did":"HJUyRbeMTesrgShkNqbAYP","verkey":"9tKZRphq5Jwoo79CVnTEv75fXQ3BQzBqFquWGARM9bUx"}
```
3. Visited the VON webserver UI located at `http://localhost:9000/` and performed the following steps in the "Authenticate a New DID" section of the UI:
  - Changed "Register from seed" to "Register from DID"
  - Populated the DID field with the value `HJUyRbeMTesrgShkNqbAYP`
  - Populated the Verkey field with the value `9tKZRphq5Jwoo79CVnTEv75fXQ3BQzBqFquWGARM9bUx`
  - Clicked the "Register DID" button

4. Clicked the "Domain" link under the "Ledger State" section of the UI
5. The ledger shows a new entry with the following data displayed under the "Transaction" header of the new entry
```
Type: NYM
Nym: HJUyRbeMTesrgShkNqbAYP
Role: ENDORSER
Verkey: 9tKZRphq5Jwoo79CVnTEv75fXQ3BQzBqFquWGARM9bUx
```

The Node.js mediator is now successfully registered as an Endorser on the ledger with permission to write new entries to the ledger.

Next the mediator must create and register a new schema and credential definition on the ledger.

1. Created a function in `/server/afj/mediator.ts` to register the schema
```typescript
const registerSchema = (agent: Agent) =>
  agent.ledger.registerSchema({
    name: `Person`,
    version: '1.0.0',
    attributes: ['Name', 'Phone'],
  })
```
2. Created a function in `/server/afj/mediator.ts` to register the credential definition
```typescript
const registerCredentialDefinition = (agent: Agent, schema: Schema) =>
  agent.ledger.registerCredentialDefinition({
    schema,
    tag: 'latest',
    supportRevocation: true,
  })
```
3. Created a method in `/server/afj/mediator.ts` to call the `registerSchema()` and `registerCredentialDefinition()` functions
```typescript
httpInboundTransport.app.get('/createschema', async (req, res) => {
  const schema = await registerSchema(agent)
  const credentialDefinition = await registerCredentialDefinition(agent, schema)  
  res.send('schema and credential definition registered')
})
```
4. Visited `http://192.168.1.72:3001/createschema` in a browser
5. Visited the VON webserver UI located at `http://localhost:9000/` and clicked the "Domain" link under the "Ledger State" section of the UI
6. The ledger shows two new entries

The first new entry is for the schema and contains the following information under the "Transaction" header
```
Type: SCHEMA
Schema name: Person
Schema version: 1.0.0
Schema attributes:
• Name
• Phone
```

The first new entry on the ledger has a "Transaction ID" value of `HJUyRbeMTesrgShkNqbAYP:2:Neighbor:1.0.0` under its "Message Wrapper" section. This value is the schema ID.

The second new entry is for the credential definition and contains the following information under the "Transaction" header
```
Type: CRED_DEF
Reference: 7
Signature type: CL
Tag: latest
Attributes:
• master_secret
• name
• phone
```

The second new entry on the ledger has a "Transaction ID" value of `HJUyRbeMTesrgShkNqbAYP:3:CL:7:latest` under its "Message Wrapper" section. This value is the credential definition ID.

The schema and credential definition are now successfully registered on the ledger.

### Issuing credentials from the Node.js mediator to the React Native edge agents
1. Created a method in `/server/afj/mediator.ts` to offer a credential based on the newly created credential definition with ID `HJUyRbeMTesrgShkNqbAYP:3:CL:7:latest` to a specific connection by supplying the following query parameters:
 - connection: the ID of the connection to offer the credential to
 - name: the value to assign to the "Name" attribute of the credential being offered
```typescript
httpInboundTransport.app.get("/offer", async (req, res) => {
  var conn = req.query.connection;

  var definitionID = "HJUyRbeMTesrgShkNqbAYP:3:CL:7:latest";

  const credentialPreview = CredentialPreview.fromRecord({
    Name: req.query.name,
  });

  await agent.credentials.offerCredential(conn, {
    credentialDefinitionId: definitionID,
    preview: credentialPreview,
  });

  res.send("credential offered");
});

```
2. Added a method to the `/server/afj/mediator.ts` file to list all connections to the mediator as links to the `/offer` method with the connection's ID as the `connection` query parameter and a shortened version of the agent's label as the `name` query parameter
```typescript
httpInboundTransport.app.get("/offerlist", async (req, res) => {
  var connections = await agent.connections.getAll();

  var conns = [];

  connections.forEach((c, i) => {
    conns.push({
      label: c.theirLabel,
      id: c.id,
    });
  });

  var list = "";

  conns.forEach((conn, i) => {    
    // use only the last part of the connection label as the "name" query parameter to be used as the "Name" attribute when issuing the credential
    // "Private Wallet Alice" becomes "Alice" and "Private Wallet Bob" becomes "Bob"
    var shortName = conn.label.split(' ')[2]
    
    // create a link to the /offer method with the connection ID as the "connection" query parameter and the shortened label as the "name" query parameter
    list = list + `<a href="http://192.168.1.72:3001/offer?connection=` + conn.id + `&name=`+shortName+`">` + conn.label + `</a><br />`
  });

  res.send(list);
});
```
3. Added `CredentialEventTypes` to the list of imports for the React Native edge agent
```javascript
import { CredentialEventTypes } from "@aries-framework/core";
```
4. Added a listener to the edge agent that will listen for credential offers and accept them
```javascript
agent.events.on(
  CredentialEventTypes.CredentialStateChanged,
  async ({payload}: CredentialStateChangedEvent) => {
    if (payload.credentialRecord.state === CredentialState.OfferReceived) {
      agent.credentials.acceptOffer(payload.credentialRecord.id);      
    }
  },
);
```
5. Visited `http://192.168.1.72:3001/offerlist` to display a list of the mediator's connections as links to the `/offer` method resulting in the following output
```
<a href="http://192.168.1.72:3001/offer?connection=4b0f8b90-058b-4fd1-9b2e-26faa635bae5&name=Alice">Issue credential to Private Wallet Alice</a>
<a href="http://192.168.1.72:3001/offer?connection=003f038c-576a-4ab6-bc9e-c3bd458384cc&name=Bob">Issue credential to Private Wallet Bob</a>
```
6. Clicked the link titled "Issue credential to Alice" to call the `/offer` method and issue a credential to Alice's agent with the "Name" attribute having a value of "Alice"
7. Imported the Buffer library to decode the credential data which is provided in base64
```javascript
import { Buffer } from 'buffer';
```
8. Created a method in `/server/mywallet/App.js` of the React Native app to decode any credentials the agent has received and log them to the console
```javascript
const showCredentials = async () => {
  var credentials = await agent.credentials.getAll();

  credentials.forEach((c, i) => {
    if (c.state === 'done') {
      var s = Buffer.from(
        c.credentialMessage.credentialAttachments[0].data.base64,
        'base64',
      ).toString();
      var sp = JSON.parse(s);
      console.log(JSON.stringify(sp.values, null, 2));
    }
  });

  return false;
};
```
9. Created a button to call the `showCredentials()` method
```javascript
<View>
  <Button
    onPress={() => {
      showCredentials();
    }}
    title="Show Credentials"
  />
</View>
```
10. Pressed the "Show Credentials" button on Alice's agent to log the credential received from the mediator to the console resulting in the following output in the console
```json
{
  "schema_id": "HJUyRbeMTesrgShkNqbAYP:2:Person:1.0.0",
  "cred_def_id": "HJUyRbeMTesrgShkNqbAYP:3:CL:7:latest",
  "rev_reg_id": null,
  "values": {
    "Name": {
      "raw": "Alice",
      "encoded": "49134673262984350012203662936972218187780495801618915546443112853445148770448"
    }
  },
  "signature": {
    "p_credential": {
      "m_2": "76092918671082413225206030085402336436948677394289535359059308875779241743382",
      "a": "10423598576783233265270818089957266406991114630011914353583266141408488991894647945752982508933783796949169509843591780929535780819619194812735176485369151515807341173514262541798505127910940009944844767491139121412767402761133032911229326871289208700364209836853327919536420064125060049912518767595324239371736669060222086085331830954339396180867530055976844001861270821405288100540643093466496589705102304146934651831215596407338087822828411757276870056477606949302739956154301273379177379917153098374681628245718608288944517279488445313736909681346493962261130541245467992909835209086231008115370704281325330918825",
      "e": "259344723055062059907025491480697571938277889515152306249728583105665800713306759149981690559193987143012367913206299323899696942213235956742930076853569989116441333963922006014813",
      "v": "5220185797045082548314232643500235417996463517433225198460165875693179296576285879219216410847081406095193848339501569049134435400099326154323123894037272866740279248935600732380800492678910869510961744421687244711203202543613005729237703834091168978421413308884686493953804126946690636665971013617689934765226516320115208625820176544759291198337391653211132083794675253250671299749030178402766654457984549542202573287963303145088719151432916828992472098783742560673341581521665459561407198660441420947248958804708836806564801249440485158423606207128312775842437255538446147107035260419629257122542841896696586001251492168978175447029368613220771569786144373979537979848545803425589258086097921284870876652639117534024014438879541125215226022969347562299852323455218351949630537433167024751657672908556710616811109998499"
    },
    "r_credential": null
  },
  "signature_correctness_proof": {
    "se": "13832383249391622270697153024037102999325064059393813703137547000669371745548580217683165507738473366773591663799660304141541083591009553188032105212220578272890863069531526807379620154661445412755700440071447592871454415264801496507310982907084800277324805256716882791567315366689149543165026886112829706479891873023365411826129422764118954398088622890964848647139086439736156494300340386976715499174197711943847811055584824878345916377426092707005438272124999831720640383454745536251062640017881724132418681520850102476293953527104432778744712937740500340950900126763991273163515998138950429572173044115915387983988",
    "c": "50780147548059214428577806855475787157956488583197774544606194457608437769429"
  },
  "rev_reg": null,
  "witness": null
}
```
11. The credential that was logged to the console contains the following keys that match the schema ID and credential definition ID
```json
"schema_id": "HJUyRbeMTesrgShkNqbAYP:2:Person:1.0.0",
"cred_def_id": "HJUyRbeMTesrgShkNqbAYP:3:CL"
```
12. The credential that was logged to the console contains a key called "values" that contains an object with a key called "Name" with a raw value of "Alice"

Alice has successfully been issued a credential by the Node.js mediator for the credential definition with ID of `HJUyRbeMTesrgShkNqbAYP:3:CL` containing a value of "Alice" for the "Name" attribute.

### Make a request from one edge agent to the other edge agent asking for proof of a credential
1. Created a `requestProof()` method in the `/server/mywallet/App.js` file to request proof of a credential from a connection with "Name" as the only requested attribute
```javascript
const requestProof = async conn => {
  var pr = await agent.proofs.requestProof(
    conn,
    {
      requestedAttributes: {person_name: {name: 'Name'}},
    },
    {
      autoAcceptProof: true,
    },
  );  
};
```
2. Added a "Request Proof" button to each connection displayed in the FlatList to call the `requestProof()` method with the ID of that connection 
```javascript
<FlatList
    data={myConnections}
    renderItem={({item}) => (
    <View style={{flexDirection: 'row', marginBottom: 20}}>    
        <Button
          onPress={() => {
            sendMessage(item.id,message);
          }}
          title="Send Message"
        />
        <Button
          onPress={() => {
            requestProof(item.id);
          }}
          title="Request Proof"
        />
        <Text>{item.name}</Text>
    </View>
    )}
    keyExtractor={item => item.id}
/>
```
3. Added `ProofEventTypes` to the list of imports for the React Native edge agent
```javascript
import { ProofEventTypes } from "@aries-framework/core";
```
4. Added a listener to `/server/mywallet/App.js` to accept incoming proof requests received by the edge agent and present proof to the edge agent that requested the proof
```javascript
agent.events.on<ProofStateChangedEvent>(
  ProofEventTypes.ProofStateChanged,

  async (event) => {
    var payload = event.payload;

    if (payload.proofRecord.state === "request-received") {
      const creds = await agent.proofs.getRequestedCredentialsForProofRequest(
        payload.proofRecord.id
      );

      const automaticRequestedCreds = await agent.proofs.autoSelectCredentialsForProofRequest(creds);

      await agent.proofs.acceptRequest(
        payload.proofRecord.id,
        automaticRequestedCreds
      );
    }
  }
);

```
5. Pressed the "Request Proof" button on Bob's agent next to the "Private Wallet Alice" connection
6. A proof request is created and sent to Alice's agent
7. An error occurs on Alice's agent

This is where the expected functionality fails. Everything up until this point has worked flawlessly. It is only when trying to create the proof presentation from Alice's agent that an error occurs.

The error in question reads as follows:
```
Possible Unhandled Promise Rejection (id: 0):
[IndySdkError: IndyError(CommonInvalidStructure): CommonInvalidStructure]
```

### Debugging considerations
The error is ambiguous and lacks verbosity.

Line 231 of [/packages/core/src/modules/proofs/ProofsModule.ts](https://github.com/hyperledger/aries-framework-javascript/blob/main/packages/core/src/modules/proofs/ProofsModule.ts#L231) from the aries-framework-javascript repository make a call to the `createPresentation()` method of the ProofService

Line 418 of [/packages/core/src/modules/proofs/services/ProofService.ts](https://github.com/hyperledger/aries-framework-javascript/blob/main/packages/core/src/modules/proofs/services/ProofService.ts#L418) from the aries-framework-javascript repository contains the following code
```typescript
this.logger.debug(`Creating presentation for proof record with id ${proofRecord.id}`)
```

When the error occurs, the console log on Alice's agent shows `Creating presentation for proof record with id...` with the correct proof record ID, so the ProofService call from the ProofsModule executes correctly through line 418 of the ProofService.

The actual error seems to occur at Line 438 of [/packages/core/src/modules/proofs/services/ProofService.ts](https://github.com/hyperledger/aries-framework-javascript/blob/main/packages/core/src/modules/proofs/services/ProofService.ts#L438) from the aries-framework-javascript repository when calling the `createProof()` method.

Any code after line 438 in the ProofService is not executed and any code after line 231 of the ProofsModule is not executed.

The error seems to be happening in the `createProof()` method from ProofService.ts
```typescript
private async createProof(
  proofRequest: ProofRequest,
  requestedCredentials: RequestedCredentials
): Promise<IndyProof> {
  const credentialObjects = [
    ...Object.values(requestedCredentials.requestedAttributes),
    ...Object.values(requestedCredentials.requestedPredicates),
  ].map((c) => c.credentialInfo)

  const schemas = await this.getSchemas(new Set(credentialObjects.map((c) => c.schemaId)))
  const credentialDefinitions = await this.getCredentialDefinitions(
    new Set(credentialObjects.map((c) => c.credentialDefinitionId))
  )

  const proof = await this.indyHolderService.createProof({
    proofRequest: proofRequest.toJSON(),
    requestedCredentials: requestedCredentials.toJSON(),
    schemas,
    credentialDefinitions,
  })

  return proof
}
```
### Debugging assumptions
I have a feeling that the issue is arising from not having a revocation registry entry defined for the credential that was issued.

I believe that I need to create an Indy Tails Server and use it as the revocation registry for the verifiable credentials that the Node.js mediator issues so that the created proof can include information about whether or not the verifiable credential has been revoked.

The documentation for requesting, presenting, and verifying proofs is scarce and I am not sure that I can resolve this issue without some guidance.

### Comments
This error is occuring at the final step of the implementation.

If the proof presentation can be sucessfully created by the holder then it will be sent to the verifier as an outbound message where it will be verified and an event listener on the verifier can handle the emitted verification event.

Once verified, acknowledgement of the verification will be sent from the verifier back to the holder where an event listener can handle the emitted acknowledgement event.

The sending of the proof presentation to the verifier happens automatically as part of the `acceptRequest()` method of the ProofService after the proof has been created.

I have not been able to get past this error to explore being able to respond to the presentation by the holder with acknowledgement from the verifier.

This is the final step of the entire implementation and I am unable to continue until this error is resolved.

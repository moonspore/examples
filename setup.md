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
```
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
Migrate Wallet Alice
Migrate Wallet Bob
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
```
{
  "@type": "https://didcomm.org/connections/1.0/invitation",
  "@id": "583ac34f-07cc-461d-96a4-34dfa3647186",
  "label": "Migrate Wallet Alice",
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
1. Added a method to the React Native app to display a list of connections
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
The two React Native edge agents have connected succesfully using the invitation that was created by Alice's Agent and pasted into Bob's agent.

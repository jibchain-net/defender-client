# Defender Sentinel Client

Defender Sentinel allows you to monitor transactions by defining conditions on events, functions, and transaction parameters, and notifying via email, slack, telegram, discord, Autotasks, and more.

Further information can be found on the OpenZeppelin documentation page: https://docs.openzeppelin.com/defender/sentinel

## Install

```bash
npm install defender-sentinel-client
```

```bash
yarn add defender-sentinel-client
```

## Usage

Start by creating a new _Team API Key_ in Defender, and granting it the capability to manage sentinels. Use the newly created API key to initialize an instance of the Sentinel client.

```js
const { SentinelClient } = require('defender-sentinel-client');
const client = new SentinelClient({ apiKey: API_KEY, apiSecret: API_SECRET });
```

### List Sentinels

To list existing sentinels, you can call the `list` function on the client, which returns a `CreateSentinelResponse[]` object:

```js
await client.list();
```

### Create a Notification

A sentinel requires a notification configuration to alert the right channels in case an event is triggered.
In order to do so, you can either use an existing notification ID (from another sentinel for example), or create a new one.

The following notification channels are available:

- email
- slack
- discord
- telegram
- datadog

The `createNotificationChannel` function requires the `NotificationType` and `NotificationRequest` parameters respectively, and returns a `NotificationResponse` object.

```js
const notification = await client.createNotificationChannel('email', {
  name: 'MyEmailNotification',
  config: {
    emails: ['john@example.com'],
  },
  paused: false,
});

const notification = await client.createNotificationChannel('slack', {
  name: 'MySlackNotification',
  config: {
    url: 'https://slack.com/url/key',
  },
  paused: false,
});

const notification = await client.createNotificationChannel('telegram', {
  name: 'MyTelegramNotification',
  config: {
    botToken: 'abcd',
    chatId: '123',
  },
  paused: false,
});

const notification = await client.createNotificationChannel('discord', {
  name: 'MyDiscordNotification',
  config: {
    url: 'https://discord.com/url/key',
  },
  paused: false,
});

const notification = await client.createNotificationChannel('datadog', {
  name: 'MyDatadogNotification',
  config: {
    apiKey: 'abcd',
    metricPrefix: 'prefix',
  },
  paused: false,
});
```

You can also list existing notification channels:

```js
const notificationChannels = await client.listNotificationChannels();
const { notificationId, type } = notificationChannels[0];
```

This returns a `NotificationResponse[]` object.

### Create a Sentinel

To create a new sentinel, you need to provide the network, name, pause-state, conditions, alert threshold and notification configuration. This request is exported as type `CreateSentinelRequest`.

An example is provided below. This sentinel will be named `MyNewSentinel` and will be monitoring the `renounceOwnership` function on the `0x0f06aB75c7DD497981b75CD82F6566e3a5CAd8f2` contract on the Rinkeby network.
The alert threshold is set to 2 times within 1 hour, and the user will be notified via email.

```js
const requestParameters = {
  network: 'rinkeby',
  // optional
  confirmLevel: 1, // if not set, we pick the blockwatcher for the chosen network with the lowest offset
  name: 'MyNewSentinel',
  address: '0x0f06aB75c7DD497981b75CD82F6566e3a5CAd8f2',
  abi: '[{"inputs":[],"stateMutability":"nonpayable","type":"constructor"},{...}]',
  // optional
  paused: false,
  // optional
  eventConditions: [],
  // optional
  functionConditions: [{ functionSignature: 'renounceOwnership()' }],
  // optional
  txCondition: 'gasPrice > 0',
  // optional
  autotaskCondition: '3dcfee82-f5bd-43e3-8480-0676e5c28964',
  // optional
  autotaskTrigger: undefined,
  // optional
  alertThreshold: {
    amount: 2,
    windowSeconds: 3600,
  },
  // optional
  alertTimeoutMs: 0,
  notificationChannels: [notification.notificationId],
};
```

Once you have these parameters all setup, you can create a sentinel by calling the `create` function on the client. This will return a `CreateSentinelResponse` object.

```js
await client.create(requestParameters);
```

If you wish to trigger the sentinel based on additional events, you could add another `EventCondition` or `FunctionCondition` object, for example:

```js
functionConditions: [{ functionSignature: 'renounceOwnership()' }],
eventConditions: [
  {
    eventSignature: "OwnershipTransferred(address,address)",
    expression: "\"0xf5453Ac1b5A978024F0469ea36Be25887EA812b5,0x6B9501462d48F7e78Ba11c98508ee16d29a03412\""
  }
]
```

You could also apply a transaction condition by modifying the `txCondition` property:
Possible variables: `value`, `gasPrice`, `gasLimit`, `gasUsed`, `to`, `from`, `nonce`, `status` ('success', 'failed' or 'any'), `input`, or `transactionIndex`.

```js
txCondition: 'gasPrice > 0',
```

Additionally, the sentinel could invoke an autotask to further evaluate. Documentation around this can be found here: https://docs.openzeppelin.com/defender/sentinel#autotask_conditions.

```js
// If other conditions match, the sentinel will invoke this autotask to further evaluate.
autotaskCondition: '3dcfee82-f5bd-43e3-8480-0676e5c28964',
// Define autotask within the notification configuration
autotaskTrigger: '1abfee11-a5bc-51e5-1180-0675a5b24c61',
```

### Retrieve a Sentinel

You can retrieve a sentinel by ID. This will return a `CreateSentinelResponse` object.

```js
await client.get('8181d9e0-88ce-4db0-802a-2b56e2e6a7b1');
```

### Update a Sentinel

To update a sentinel, you can call the `update` function on the client. This will require the sentinel ID and a `CreateSentinelRequest` object as parameters:

```js
await client.update('8181d9e0-88ce-4db0-802a-2b56e2e6a7b1', requestParameters);
```

### Delete a Sentinel

You can delete a sentinel by ID. This will return a `DeletedSentinelResponse` object.

```js
await client.delete('8181d9e0-88ce-4db0-802a-2b56e2e6a7b1');
```

### Pause and unpause a Sentinel

You can pause and unpause a sentinel by ID. This will return a `CreateSentinelResponse` object.

```js
await client.pause('8181d9e0-88ce-4db0-802a-2b56e2e6a7b1');
await client.unpause('8181d9e0-88ce-4db0-802a-2b56e2e6a7b1');
```

### Failed Requests

Failed requests might return the following example response object:

```js
{
  response: {
    status: 404,
    statusText: 'Not Found',
    data: {
      message: 'subscriber with id 8181d9e0-88ce-4db0-802a-2b56e2e6a7b1 not found.'
    }
  },
  message: 'Request failed with status code 404',
  request: {
    path: '/subscribers/8181d9e0-88ce-4db0-802a-2b56e2e6a7b1',
    method: 'GET'
  }
}
```
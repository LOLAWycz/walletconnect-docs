# Wallet Usage

## Initialization

To create an instance of Web3Wallet, you need to pass in the `core` and `metadata` parameters.

```dart
Web3Wallet web3Wallet = await Web3Wallet.createInstance(
  core: Core(
    relayUrl: 'wss://relay.walletconnect.com', // The relay websocket URL
    projectId: '123',
  ),
  metadata: PairingMetadata(
    name: 'Wallet (Responder)',
    description: 'A wallet that can be requested to sign transactions',
    url: 'https://walletconnect.com',
    icons: ['https://avatars.githubusercontent.com/u/37784886'],
  ),
);

```

## Chain Configuration

Set up the methods and chains that your wallet supports.

```dart
web3Wallet.registerRequestHandler(
  namespace: 'kadena',
  method: 'kadena_sign',
);
```

## Pairing

Scan the QR code and parse the URI, and pair with the dapp. Upon the first pairing, you will immediately receive `onSessionProposal` and `onAuthRequest` events.

```dart
Uri uri = Uri.parse(scannedUriString);
await web3Wallet.pair(uri: uri);
```

## Responding to a Session Proposal

Set up the proposal handler that will display the proposal to the user after the URI has been scanned.

```dart
late int id;
web3Wallet.onSessionProposal.subscribe((SessionProposal? args) async {
  // Handle UI updates using the args.params
  // Keep track of the args.id for the approval response
  id = args!.id;
})
```

## Responding to Request

```dart
web3Wallet.onSessionRequest.subscribe((SessionRequestEvent? request) async {
  // You can respond to requests in this manner
  await clientB.respondSessionRequest(
    topic: request.topic,
    response: JsonRpcResponse<String>(
      id: request.id,
      result: 'Signed!',
    ),
  );
});
```

## Auth Handling

Set up the auth handling.

```dart
clientB.onAuthRequest.subscribe((AuthRequest? args) async {
    // This is where you would
    // 1. Store the information to be signed
    // 2. Display to the user that an auth request has been received

    // You can create the message to be signed in this manner
    String message = clientB.formatAuthMessage(
        iss: TEST_ISSUER_EIP191,
        cacaoPayload: CacaoRequestPayload.fromPayloadParams(
        args!.payloadParams,
        ),
    );
});
```

## Approving the Sign Request

Present the UI for approval.

```dart
final walletNamespaces = {
    'eip155': Namespace(
        accounts: ['eip155:1:abc'],
        methods: ['eth_signTransaction'],
    ),
    'kadena': Namespace(
        accounts: ['kadena:mainnet01:abc'],
        methods: ['kadena_sign_v1', 'kadena_quicksign_v1'],
        events: ['kadena_transaction_updated'],
    ),
}
await web3Wallet.approve(
    id: id,
    namespaces: walletNamespaces // This will have the accounts requested in params
);
```

## Rejecting the Sign Request

To reject the request, pass in an error code and reason. They can be found [here](https://docs.walletconnect.com/2.0/specs/clients/sign/error-codes).

```dart
await web3Wallet.reject(
    id: id,
    reason: ErrorResponse(
        code: 4001,
        message: "User rejected request",
    ),
);
```

## Approving an Auth Request

You can approve a dapp's auth request by responding with the user's signature.

```dart
String sig = 'your sig here';
await web3Wallet.respondAuthRequest(
  id: args.id,
  iss: 'did:pkh:eip155:1:0x06C6A22feB5f8CcEDA0db0D593e6F26A3611d5fa',
  signature: CacaoSignature(t: CacaoSignature.EIP191, s: sig),
);
```

## Rejecting an Auth Request

To reject the request, pass in an error code and reason. They can be found [here](https://docs.walletconnect.com/2.0/specs/clients/sign/error-codes).

```dart
await web3Wallet.respondAuthRequest(
  id: args.id,
  iss: 'did:pkh:eip155:1:0x06C6A22feB5f8CcEDA0db0D593e6F26A3611d5fa',
  error: WalletConnectErrorResponse(code: 12001, message: 'User rejected the signature request'),
);
```

# To Test

Run tests using `flutter test`.
Expected flutter version is: >`3.3.10`

# Useful Commands

* `flutter pub run build_runner build --delete-conflicting-outputs` - Regenerates JSON Generators
* `flutter doctor -v` - get paths of everything installed.
* `flutter pub get`
* `flutter upgrade`
* `flutter clean`
* `flutter pub cache clean`
* `flutter pub deps`
* `flutter pub run dependency_validator` - show unused dependencies and more
* `dart format lib/* -l 120`
* `flutter analyze`
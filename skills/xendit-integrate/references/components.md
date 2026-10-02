# Xendit Components

Xendit Components refers to 3 separate libraries depending on your user's platform.

- https://github.com/xendit/xendit-components-web - install using npm
- https://github.com/xendit/xendit-components-ios - install using Swift Package Manager
- https://github.com/xendit/xendit-components-android - install from Maven Central

Use the documentation in the README.md of the respective repo.

IMPORTANT: always listen to and handle error events.

## components_configuration

When creating a components session, you must specify the domains or apps that are allowed to access it.

- To specify a web page: `components_configuration: { origins: ["https://checkout.mycompany.com"] }`, it must exactly match the origin of your page i.e. no slash (localhost is allowed, port numbers are allowed, non-https is not allowed, wildcards are not allowed)
- To specify an android app: `components_configuration: { origins: ["android:com.mycompany.myapp"] }`
- To specify an iOS app: `components_configuration: { origins: ["ios:com.mycompany.myapp"] }`

On web, you must also specify `components_configuration.return_url` (if you don't the user will land on a generic status page). `return_url` should direct the user back to the same flow they were on. It's used when returning from redirect payments.

## Web-specific tips

### Always listen to fatal-error

Fatal error indicates you can no longer use the XenditComponents instance and must back out of the payment flow. Ignoring this error breaks the UX. Ensure you print errors to the console.

### Always attach event listeners first

Some events fire synchronously. Attaching an event after calling a function e.g. `submit()` will not work because the event has already fired.

```
// ❌ incorrect, submission-begin fires before listener is attached
components.submit();
components.addEventListener("submission-begin", showPendingState);
```

### Always call abortSubmission if user cancels payment

Failing to call `abortSubmission` when you back out of an in-progress payment (between submission-begin and submission-end) will leak a connection. Call `abortSubmission` to close the connection.

### Component type selection

The Components SDK on web supports a payment method picker component and individual payment channel components.

Coding agents should determine which the user wants before beginning. Users with existing custom channel pickers will need to use individual components to integrate with their custom channel picker.

### Searching for payment methods

If you want to reference a specific payment method, use `getActiveChannels({filter: channelCode})[0]`. ChannelCode can be a string, regex or array of strings. You can also fetch everything and manually filter.

### Instant submit for QR

You can submit some payment methods (e.g. QRs) without waiting for user input.

Note you cannot call submit from the init event, and if you call submit from a timeout you must clean it up.

If only showing the QR flow, the end of the flow is the submission-end event.

### Action flow

Actions (3DS iframes, QRs, etc...) are opened in a modal window by default.

Coding agents should establish if the user has an existing action flow and integrate with it by using `createActionContainerComponent` and mounting it in the existing action flow.

### Default action modal z-index

If you don't use `createActionContainerComponent`, a modal window will be created with an action container for you.

It has no z-index (it has position:fixed, inserted at end of document). If you have elements with z-index you must override the z-index of `.xendit-dialog-backdrop` to have a higher z-index.

### GooglePay development

GooglePay will not work in development unless you're using a trusted cert. (Use `chrome://certificate-manager/` to trust a cert). If you _EVER_ ignore a certificate error, GooglePay will not work until the browser is restarted.

## React-specific tips

### Match XenditComponents lifecycle to react component lifecycle

Store the XenditComponents instance in a ref. It is much easier than storing it in a state.

Make a component that creates a XenditComponents instance when mounted and destroys it (aborts) when unmounted.

### Attach event listeners in effects

If event listeners have dependencies, you can decouple them from the constructor by giving them their own hook. Only make extra hooks like this if necessary.

```
  useLayoutEffect(() => {
    if (!sdkRef.current) return;

    function onSuccess() {
        // do something with deps
    }

    sdkRef.current?.addEventListener("session-complete", onSuccess);
    return () => sdkRef.current?.removeEventListener("session-complete", onSuccess);
  }, [deps]);
```

### Beware effect order

Ensure the XenditComponents constructor runs in the first effect hook.

Prefer useLayoutEffect for this use case. Never mix useEffect and useLayoutEffect as it makes the effect order weird.

IMPORTANT: Strict mode causes effects to run twice. Do not let it initialize XenditComponents twice. If the ref already exists, reuse it.

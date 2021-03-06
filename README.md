# better-client-side-react
How to have a control room for Serverless Apps deployments?

This is part of a blog post I'm writing on medium. But the basic idea is on a servelrless React App which is completnly rendered on client side we don't have that much control over deployments and what needs to happen after, before. Some of the challenges are:
- If the user is using the App and there is a new version deployed how can we refresh the page for user so they get the latest version?
- If we need a maintenance window to run something on our API service or there is DB migration running that for some reasons we need to put the App on maintenance window, how can we do it?

The overall idea is we can create a small WebSocket server that communicates these commands with our Frontend Apps. For example if our deployment workflow is push the code to AWS S3 ---> Invalidate the Cache on CLoudFront ----> After the invalidation is done, send a __message__ to all clients to let them there is a new version and they need to refresh their page to get the latest __index.html__ (which is the entry point for our latest version)

On the client side we'll listen to these commands from our WebSocket and take the appropriate actions:
```
socket.current.onmessage = function (event) {
      const data = JSON.parse(event.data);
      switch (data.messageId) {
        case "new-version":
          notify.show(
            <div>
              <h2>Sorry to intrupt you 🙈</h2>
              <p>
                A new version of our App is available and to get the latest
                features you should refresh your page
              </p>
              <button onClick={() => window.location.reload()}>Reload</button>
            </div>,
            "warning",
            -1
          );

          break;
        case "maintanace":
          if (data.status === "on") {
            notify.show(
              <div>
                <h2>Sorry to intrupt you 🙈</h2>
                <p>{data.message}</p>
                <button onClick={() => notify.hide()}>Ok</button>
              </div>,
              "warning",
              -1
            );
            setMaintWindow(true);
          } else {
            setMaintWindow(false);
          }
          break;
        default:
          break;
      }
    };

```

# Setup on your local machine
1. Clone the repo  `git clone https://github.com/mostafa-drz/better-client-side-react.git `
2. run `yarn dev` from root
It opens the client on port `3001`, Express/Web socket Server on `8080

The Express App is simply responsible for listneing to commands from our pipelines/backend/other tooling and then the websocket server broadcats that command.

__NOTE__ None of the codes in this repository are production ready, for example there is no auth checking on the Express App, the purpose of this repo is just to demonstracte a paradigm that can be used in real Apps in production.
You can read more about it here: https://darehzereshki-mostafa.medium.com/a-control-room-for-client-side-rendered-apps-31047e011a37

Stay safe 🤿

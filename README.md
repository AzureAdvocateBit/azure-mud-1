# Yet Another Browser Mud
![Linting and Validation Checks](https://github.com/lazerwalker/azure-mud/workflows/Linting%20and%20Validation%20Checks/badge.svg) [![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Flazerwalker%2Fazure-mud%2Fmain%2Fserver%2Ftemplate.json)

This is a playful text-based online social chat space. You can think of it as a hybrid between communication apps like Slack and Discord and traditional text-based online game spaces such as MUDs and MOOs.

It was primarily being built for [Roguelike Celebration 2020](https://roguelike.club), but can hopefully be repurposed for other events or communities.

On the backend, it's powered by a serverless system made up of Azure Functions, Azure SignalR Service, and a Redis instance (currently provided by Azure Cache for Redis).

On the frontend, it's a rich single-page webapp built in TypeScript and React, using the Flux architecture via the `useContext` React hook.

[This article](https://dev.to/lazerwalker/using-game-design-to-make-virtual-events-more-social-24o) provides some insight into the design principles underlying this project.

## Setting up a development environment

### Frontend Dev

1. Clone this repo

2. Run `npm install`

3. `npm run dev` will start a local development environment (at `http://localhost:1234/index.html` by default). It auto-watches changes to HTML, CSS, and JS/TS code, and attempts to live-reload any connected browser instances.

4. `npm run build` will generate a bundled version of the webapp for distribution. 

This repo is set up to automatically deploy to GitHub Pages when code is merged into master (see ""Deploying New Changes via GitHub Actions" below for how to finish configuring that for your fork). If you want to host your frontend elsewhere, that's totally fine too! You can serve the asset bundle generated by `npm run build` anywhere, although it will need to be served over SSL/HTTPS.

### Backend Dev

After cloning this repo, the `server` directory contains all of the backend code. You may want to run `npm install` within the `server` directory to pull in dependencies for IDE autocompletion and such. 

However, you cannot actually run the backend locally. You'll need to deploy your own server instance of the backend to test changes.

## Deploying This Project
Currently, this project only runs on Azure. This requires your own [Azure subscription](https://azure.com/free/?WT.mc_id=spatial-8206-emwalker). 

If you don't already have an Azure account/subscription, you'll get a few hundred bucks of credits to use your first month, but if that's not the case you will want to keep an eye on the fact that **running this backend will cost you actual money**. 

### Deploying via ARM Template

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Flazerwalker%2Fazure-mud%2Fmain%2Fserver%2Ftemplate.json)


The easiest way to deploy a backend is to use the template we have prepared. Going to [this link](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Flazerwalker%2Fazure-mud%2Fmain%2Fserver%2Ftemplate.json) will allow you to deploy a server backend to an Azure resource group you specify that will have everything configured, but won't actually contain code yet.

There are still a few things you need to manually configure before the app will function.

1. In order for users to log in, you will need to configure Twitter and/or Google authentication. Follow [these](https://docs.microsoft.com/en-us/azure/app-service/configure-authentication-provider-twitter?WT.mc_id=spatial-8206-emwalker) instructions for Twitter, and [these](https://docs.microsoft.com/en-us/azure/app-service/configure-authentication-provider-google?WT.mc_id=spatial-8206-emwalker) for Google.

2. You'll need to modify the frontend to actually use your new backend! In `src/config.ts`, update the hostname to point to your own Function App instance (the Azure URL for your backend — typically `https://your-project.azurewebsite.net`, where `your-project` is the project name you entered when deploying the Azure ARM template).

3. Finally, you need to actually deploy the backend code before everything will work. You have three main options (below), but after doing this you should have a working app!

#### Deploying new Changes via GitHub Actions

By default, every time code is merged into the `main` branch in this repo, both the frontend and backend are deployed. It's very little work to configure this same behavior on your GitHub fork of this project:

1. In `.github/workflows/deploy.yml`, find the "Deploy Backend" step. Replace the `app-name` with your own Azure app name, and follow [these instructions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-how-to-github-actions?WT.mc_id=spatial-8206-emwalker) to generate a publish profile and add that as a GH Secret

2. If you want to continue to use GH Pages, make sure it is enabled for your project in the GH repo Settings (serving from the root of the `gh-pages` branch). This will also work with a custom domain if properly configured. If you would like to use a different static site host, simply remove the "Deploy Frontend" step from the `.github/workflows/deploy.yml` file.

#### Deploying new Changes via VS Code

If you use VS Code as an IDE, the Azure Functions extensino makes it extremely easy to deploy directly from there. Check out [this tutorial](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-function-vs-code?pivots=programming-language-javascript?WT.mc_id=spatial-8206-emwalker) for more info.

#### Deploying new Changes via CLI

If you have the [Azure Functions Core Tools](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?WT.mc_id=spatial-8206-emwalker) installed, you can run `func azure functionapp publish [project name]` to deploy directly from the CLI.


### Costs and Scaling

For development purposes, you can use the free tier of both SignalR Service and Azure functions, you just need to pay for a small Redis instance (~$15/month). As mentioned, if you're a new Azure user, you'll get more than enough free credits to cover hosting this for your first month.

To get this ready for production use, all you need to do is scale up your SignalR Service usage tier. Running this project with a single SignalR unit (good for up to 1,000 concurrent users) will cost you roughly $2.50 per day, with each additional SignalR unit (another 1,000 concurrents) adding roughly an additional $1.50 per day. These numbers are all rough ballpark figures.

Other than managing SignalR units, you won't need to worry about adjusting capacity in order to scale. Azure Functions charges you based on usage, and it's extremely unlikely you'll need to scale up Redis unless you have tens of thousands of concurrent users. At Roguelike Celebration, with an average of around 300 concurrent users, we never hit more than a few hundred kilobytes of Redis data or used even 1% of our processing power.

### Manual Deployment Instructions

If you would prefer to not use the ARM template above, here is how you can manually configure a set of Azure resources to run this project.

1. Deploy the project to a new Azure Function App instance you control. I recommend using VS Code and the VS Code Azure Functions extension. See the "Publish the project to Azure" section of [this tutorial](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-function-vs-code?pivots=programming-language-javascript?WT.mc_id=spatial-8206-emwalker) for details. You can also use the Azure CLI or any other method.

2) In the Azure Portal, sign up for a new [Azure SignalR Service](https://docs.microsoft.com/en-us/azure/azure-signalr/signalr-overview?WT.mc_id=spatial-8206-emwalker) instance. For development purposes, you can probably start with the free tier.

3) Grab the connection string from your Azure SignalR Service instance. Back in the settings for your Function App, go to the Configuration tab and add a Application Setting with the key `AzureSignalRConnectionString`

4) Set up an [Azure Cache for Redis](https://docs.microsoft.com/en-us/azure/azure-cache-for-redis/cache-overview?WT.mc_id=spatial-8206-emwalker) instance. Again, the cheapest tier is likely acceptable for testing purposes. You could theoretically use an alternative Redis provider, as nothing about our use is Azure-specific, but I have not tried this.

5) As above, you want to take your Redis access key, the hostname, and the port, and add them as Application settings to the Function App with the keys `RedisKey` and `RedisHostname`, `RedisPort`.

6) Set up Twitter authentication. In the Azure Portal, pull up the Function App and go to "Authentication". In another window, go to <https://developer.twitter.com/apps> and register a new Twitter developer application. You will need to paste the consumer key and secret from Twitter into the Azure setup screen for Twitter. The callback URL to enter in the Twitter app is `https://your-function-app.azurewebsites.net/.auth/login/twitter/callback`, swapping in the URL hostname of your Function app. In the Azure Portal authentication screen, make sure that "Token Store" is on, and add "http://localhost:1234" (and any other URLs you want to be able to use) to the Allowed External Redirect URLs list.

7) Set up CORS in the Azure Portal page for the Function app. There's a "CORS" menu item on the left. Allow `http://localhost:1234` for local development, as well as whatever URLs you're using for a production version of the frontend.

8) In `src/config.ts` in this repo, update the hostname to point to your own Function App instance (the Azure URL for your backend, NOT wherever you're hosting the app's frontend).

## Deployment and CI/CD via GitHub Actions

By default, this project uses GitHub Actions for a few things. 

The "lint" workflow runs on every open PR and on every commit to main, and checks (a) whether the code passes a TypeScript compilation/typecheck step, (b) whether all your room description links are valid (see below), and (c) whether the front-end code passes accessibility best standars (via [axe linter](https://axe-linter.deque.com/))/.

The "deploy" workflow runs every time code is pushed to the `main` branch (or a PR is merged, etc). It builds the app, and deploys both the front-end (to GitHub Pages) and the back-end (to Azure Functions).

If you fork this project, you will get this behavior for free, although you will need to change a few things:

You can naturally disable either or both of these behaviors if you'd prefer.

## Designing your own space

This project is open-source, and currently includes the scripting code for the space we used at Roguelike Celebration. However, we would ask that you design your own event space on this platform, rather than using Roguelike Celebration's space (room descriptions, map, etc). Fortunately, it's easy to customize the space!

Rooms are currently defined in `server/src/rooms`. Each room definition is a JSON object containing its description and other information; check the definitions in `server/src/rooms/index.ts` for an example there. The list of rooms in that file is definitive, but you can also see there how to define a room in an external file and link it in.

Within room descriptions, this project uses a custom Twine-like syntax for links. 

* A link to a `[[room]]` will link directly to a room whose `id` is `room`. 
* A link to `[[another room->someOtherRoom]]` will display the text "another room" but link to the room whose `id` is `someOtherRoom`
* A link that looks like it links to the `id` "item" (e.g. `[[a heavy book->item]]`) will result in the player picking up that item, so long as that item string is an exact match in the list in `server/src/allowedItems.ts`.
* Links can also trigger client-side code if the link `id` refers to a function in the client-side `src/linkActions` object. As an example, `[[Get a random food item->generateFood]]` will trigger the `generateFood` function that in turn assigns the player an inventory object that is a procedurally-generated piece of food.

There is an automated script to validate that none of your links are broken (i.e. all room links go to valid room IDs, all linked items are in the allowed item list, and all client-side functions actually exist). You can run this by typing `npm run lint-rooms` in the main project directory, and it also runs automatically on PRs if you have the default GitHub Actions running.

Links between rooms are purely visual. If an attendee is moving rooms using the map or the `/move` command, they can move to any room at any time. It's still best practice to include links to "nearby" rooms (matching your visual map) to help users navigate the space.

Right now, because room descriptions are part of the server codebase, changing room data requires redeploying the entire server backend. Changing that is a high development priority.

## Editing the map
The ASCII map was created with [MonoDraw](https://monodraw.helftone.com), a Mac-only ASCII art tool. You'll want to open the `map.monopic` file in that, export your changes, paste the ASCII string into `src/components/MapView.tsx`, and then update any changes to the two datasets of persistence identifiers and clickable areas. Note that the coordinates listed in the MonoDraw view are 1-indexed, whereas the app itself expects 0-indexed coordinates.

## Contributions

If you're looking to get involved: awesome! There's a "Good First Issue" tag in this repo's GitHub Issues that may point you towards something. If you want to work on something, it might be nice to comment that you're looking into it in case others are already working on it or were thinking about it.

Fork this repo, make your changes, open a pull request! Once you've contributed, I'm fairly liberal with granting people contributor access, but the `main` branch is still locked.

Pull requests are run through a few automated checks. If the `ESLint` checks fail, first try running `npm run eslint-fix` to try to automatically fix as many of the errors as you can; anything that doesn't catch will need to be fixed manually.

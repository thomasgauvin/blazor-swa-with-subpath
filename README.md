# Blazor Static Web Apps with sub-path

If you are looking to host a Blazor WASM application on Static Web Apps, but want your Blazor hosted with a sub-path, this is an approach to get this working.

## Disclaimer
* This project is provided under MIT license.
* This is a proof-of-concept and needs to be further developed and tested.
* This project may cause conflicts with Static Web Apps' built-in authentication and backend integration. A majority of projects should not require such an approach or sub-path workaround. 

## Getting Started

### Create your Blazor WASM application
1. Create a Blazor WASM application: `dotnet new blazorwasm -n blazor-swa-with-subpath`.
2. Open `blazor-swa-with-subpath` in VSCode.
3. You can now run the project with `dotnet run`.

### Edit your Blazor application to support a subpath such as `app`.
This is needed so that your Blazor application can make requests to the right files when it is deployed with a subpath. For this example, we will use subpath `app`.

1. In the blazor-swa-with-subpath.csproj file, add `<StaticWebAssetBasePath>app</StaticWebAssetBasePath>`. This ensures that when you run `dotnet publish`, the publish folder is created with the static web assets within an `wwwroot/app` folder.
2. In the `wwwroot/index.html` file, change the base href to `<base href="/app/" />`. This ensures that requests from your index.html are prepended to `/app/`. This will break the development website served by `dotnet run`, so consider having separate `index.html` files for development and deployment.
3. In the `wwwroot/index.html` file, change the script for blazor.webassembly.js to `<script src="/app/_framework/blazor.webassembly.js"></script>`. This will also break the development website served by `dotnet run`, so consider having separate `index.html` files for development and deployment.

    Note: You may also consider making these changes to Blazor's `index.html` after building your project with `dotnet publish` to maintain a working development experience.

4. Build your Blazor project with `dotnet publish -o publish`. This will create a `publish` folder, containing `wwwroot` folder, which contains folder `app`, which contains the Blazor application.

5. Optional step: You may now serve your built Blazor application with `dotnet serve -o --fallback-file ./app/index.html`. The fallback file indicates the entry point of your Blazor app, in case a user tries to access a specific page of your Blazor app directly. Navigate to `http://localhost:<PORT>/app` to see your built Blazor app working.

### Add the necessary Static Web Apps configurations to host your Blazor WASM app with a sub-path.
To host your Blazor WASM app with a sub-path, you must add the following configurations so that Static Web Apps redirects users to your Blazor app on the `/app` path.

The wwwroot folder will contain the following: `app` folder (contains our Blazor application), `index.html` which is needed by Static Web Apps and redirects to the Blazor app, and `staticwebapp.config.json` which contains the necessary redirects and navigation fallbacks for the Blazor app. 

1. Within `publish/wwwroot`, create an `index.html` file with the following contents:
    ```html
    <!DOCTYPE html>
    <html lang="en">

    <head>
        <meta http-equiv="Refresh" content="0; url='/app/'" />
    </head>

    <body>
    Redirecting to Blazor application
    </body>

    </html>
    ```
    This HTML is needed since Static Web Apps requires an index.html at the root of the project. It redirects requests to `/index.html` to `/app/`, where our Blazor app is hosted.

2. Within `publish/wwwroot`, create a `staticwebapp.config.json` file with the following contents:

    ```json
    {
        "navigationFallback": {
        "rewrite": "/app/index.html"
        },
        "routes": [{
            "route": "/",
            "redirect": "/app/"
        }]
    }
    ```

This provides the navigation fallback for the entry-point of the Blazor WASM application. This also redirects customers accessing `/` to `/app/` which is the entrypoint of our Blazor WASM application.

3. (Optional) Use the `swa cli` to simulate the deployed Static Web Apps experience locally. 
    1. From `publish/wwwroot`, run `swa start`. This will simulate your Static Web Apps locally, along with the `staticwebapp.config.json` and newly created `index.html file`.
    2. Open `http://localhost:4280/` in your browser. 
    ![alt text](./.readme/Screenshot%202023-03-02%20181022.png)

    Note: If this simulated Static Web Apps environment is not working, debug and resolve before moving onto the next step.

### Deploy to Static Web Apps with the SWA CLI

1. Navigate to the `/publish` directory (`cd ..`).
2. Deploy the contents of `/publish/wwwroot` using `swa deploy wwwroot`. 
    
    Note that`/publish/wwwroot` should contain the `app` folder, the `index.html` file, and the `staticwebapp.config.json` file. The publish folder has been provided in this repository for reference.

3. Browse you app at the deployed project link indicated by the `swa cli`. For instance, `https://blue-bay-088b18a1e-preview.westus2.2.azurestaticapps.net`.


![alt text](./.readme/Screenshot%202023-03-02%20182119.png)

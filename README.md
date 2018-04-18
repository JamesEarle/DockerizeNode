# DockerizeNode
Create and deploy a Docker image from your Node.js web app.

1. Create git repository
2. Make sure you have express installed globally
    a. `npm install -g express`
3. Initialize an express application inside repository
    a. `express MyAppName`

## Basic Steps ##
### Create Image ###
- [MS Documentation - Prepare App](https://docs.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-prepare-app)

First step is always the docker file. Pick a Node version you want to use. See [the Docker store](https://store.docker.com/images/node) for a list of available Node images. After that you setup the application code and working environment. We're using Node.js so we install our dependencies then run the application. You can run using `npm start`, just be sure that you've defined a `start` script in your `package.json`

```dockerfile
# Source Node image
FROM node:latest

# Create application directory
RUN mkdir -p /usr/src/app

# Copy application files
COPY ./app/ /usr/src/app/

# Set the working directory for your environment
WORKDIR /usr/src/app

# Install dependencies
RUN npm install

# Run application
CMD npm start
```

Open CLI and try `docker pull node:latest` to double check your connection. If it fails, double check network settings (secure, firewall IP, not on VPN) and double check shared drives

Ensure docker file is **not in source code directory**, it should be one level above. e.g. like below

```
Documents/
    |   Blog/
    |   MyApp/
    |   |   .gitignore
    |   |   app/
    |   |   |   bin
    |   |   |   node_modules
    |   |   |   public
    |   |   |   routes
    |   |   |   views
    |   |   |   app.js
    |   |   |   package.json
    |   |   DOCKERFILE
    |   |   README.md
    |   Othercode/

```

When you build the docker image you want to be on the **same level** as your main code folder. So in the above we would be at `C:\Users\James Earle\Documents\` (enter `pwd` in the terminal to check your current location)

```powershell
docker build ./MyApp -t myapp
```

You'll see output as Docker works through each step you defined in the docker file, and then a success message if nothing went wrong.

Test that your image works by by running it in a local container. **Be sure to expose the same port your app code exposes**

```
docker run -d -p 3000:3000 myapp
```

Now visit `localhost:3000/` (or whatever port you're using) and your application should appear.

### Create Azure Container Registry (ACR) ###
 - [MS Documentation - Azure Container Registry](https://docs.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-prepare-acr)

Login to CLI using `az login` or the portal by navigating to [portal.azure.com](portal.azure.com)

In the portal go to "New Resource" and search for Azure Container registry. Then follow on screen instructions.

```
az group create --name MyResourceGroup --location westus

az acr create -g MyResourceGroup --name MyACR --sku Basic --admin-enabled true
```

After this you login to the ACR, then you tag your image and push from docker to this ACR.

### Deploy Application
3. Deploy image to ACR
    - https://docs.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-deploy-app

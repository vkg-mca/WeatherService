- [**Hint** How to cerated markkdown file: https://pandao.github.io/editor.md/en.html]
- [**Containerization** https://www.youtube.com/watch?v=Po9jQS7WBDQ]
- [**Deployment** https://www.youtube.com/watch?v=vBx7WY25fM0]

# DEVELOPMENT

- Move to folder X:\Cloud\AZURE\WeatherService
- Execute  `git init` from terminal
- Go github. com and create the repository https://github.com/vkg-mca/WeatherService
- Go to folder WeatherService and fire command `dotnet new webapi -o WeatherService --no-https` to create an ASP.NET Web API (WeatherService)
- Execute `cd WeatherService` followed by `code .` commands to oepn in VSCode
- Add Swagger support (optional)
- Add Docker file
	- VSCode
		- Add the docker extenssion
		- Navigate to View-> Command Palette 
			- Docker: Add Docker Files to Workspace...
			- ASP.NET Core
			- Linux
			- 80
	- VisualStudio
		- Add Docker Support
- Commit the code to local branch
- Execute `git remote add origin git@github.com:vkg-mca/WeatherService.git` from terminal
- Execute `git branch -M main` from terminal
- Execute `git push -u origin main` from terminal
- Build abd debug the project to see the code running in browser, alternatively can use postman as well

# CONTAINERIZATION

- Open the Terminal and navigate to project directory
- Execute command `docker build -t vkgmca/weatherservice:v1 .` (wait to build image)
- Execute command `docker images`
- Execute command `docker run -it --rm -p 8080:80 vkgmca/weatherservice:v1`
- Go to browser and open url `http://localhost:8080/weatherforecast`
- Execute Ctrl+C from terminal to stop the container
- Execute command `docker push vkgmca/weatherservice:v1` to push the image to dockerhub container registry
- Goto `https://hub.docker.com/` and see the image available here

# DEPLOYMENT TO AKS

- Create Container Registry and an AKS cluster
	- Goto azure portal and search for `Container Registry` service and create one, name it `vkgcontainerregistry`
	- Open VSCode/Terminal and execute command `az login`, follow instructions to complete login process
	- To check the right subscription execute `az account show`
- Create `Service Principal` for service identity
	- Execute `az ad sp create-for-rbac --skip-assignment`and keep note of `appId` and `password` values as they will be required at later stage
- Goto azure portal and complete service principal deployment
	- Goto deployed container registry `vkgcontainerregistry`
	- Click `Access Control (IAM)` link and add `add role assignment` with following values
		Role = `AcrPull`, Select dropdown = Select principal searching by appId copied from above step
	- Newly added role should reflect on the screen
- Create Kubernetes service
	- Select `Azure Kubernetes` from All Services to create
	- Enter subscription and resource group name along with kubernetes cluster name `vkg-k8s-cluster`, enter rest details
	- In `Authntication` tab need to fill `Service Principal`
		- Click `Configure Service Principal`, select existing and enter below values
			- Service Principal client ID: the appId copied from previous step
			- Service principal client secret: password copied from previous step
	- Goto Terminal and login `az acr login --name vkgcontainerregistry`, result should be login succeeded
	- Execure `docker images` command to see the images available, we want to send `vkgmca/weatherservice` image to ACR
	- Execute `docker tag vkgmca/weatherservice:v1 vkgcontainerregistry.azurecr.io/weatherservice:v1` to tag current image `vkgmca/weatherservice:v1` as login server / image name
	- Execure `docker images` again and notice the newly created image exists

- Push WeatherService container to container registry
	- Execute `docker push vkgcontainerregistry.azurecr.io/weatherservice:v1`
	- Goto Azure Container Registry -> Repositories and see the newly service present there

- Generate Kubernetes yaml files to describe deployment
	- in VSCode go to extenssion and install Kubernetes extension from Microsoft
	- Add a new file `deployment.yml`, start typing deploy and click on the text appeared that creates the template
	- Add `service.yml` and start typing service and select - service allows internal and external interaction to the pod
	- 

- Deploy WeatherService to container to AKS
	- Connect to the cluster `az aks get-credentials --resource-group vkg-resource-group --name vkg-k8s-cluster`, this command will download credentials and merge into local kube config file
	- Execute `kubectl get nodes` to see the available 1 node we configured ealier
	- Execute `kubectl apply -f .\manifiest\deployment.yml`
	- Execute `kubectl get deployments`
	- Execute `kubectl get pods` -> keep checking until status turns to `Running`
	- Execute `kubectl apply -f .\manifiest\service.yml`
	- Execute `kubectl get services` -> keep checking until external IPis allocated




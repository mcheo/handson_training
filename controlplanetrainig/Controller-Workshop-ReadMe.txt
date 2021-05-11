## CONTROLLER ##

Install Pre-Req

    su - ubuntu
	sudo apt-get update

	sudo apt-get install jq -y
	sudo apt-get install socat -y
	sudo apt-get install conntrack -y

        cd /opt/
	sudo tar xvzf controller-installer-3.12.1.tar.gz

	cd controller-installer
	./install.sh

	y # Requires Docker CE to be installed ?

	y # Do you want to use an embedded DB?

	local # Provide config DB Volume type
	local # Provide time series DB volume type

	y # Do you accept the License Agreement

	SMTP Details
	example.com
	25
	n
	n
	donotreply@example.com

	ADMIN User
	10.1.1.4
	admin
	admin
	admin@f5.com
	Welcome@123
	y # Generate SSL Certificate

Update License File

Create User

Platform >> Users >> + Create
User: initial.lastname@f5.com
Password: Ensure it's a long password, example Passw0rd2021


Navigate Controller:

Install Controller Agent:
Infrastructure >> Create Instance >> Add an existing instance
Name: lb.nplus1.udf
Location: Unspecified
Check - Allow insecure server connections
Copy the CURL script and run on the adc.nplus.udf VM

### ADC Use Case ###

# Create Environment:
Navigate to Environment, click on "Create Environment" button
Environment >> Create Environment
Name: production
Display Name: Production

# Create Gateway:

Navigate to Gateways, click on "Create Gateway" button
Name: adc.nplus.udf
Display Name: adc.nplus.udf
Environment: Production
Click Next
Instances Refs: lb.nplus.udf
Click Next
Hostname: http://10.1.1.6
Click Submit

Create App:
Navigate to Apps, click on "Create App" button
Name: adc-demo
Display Name: ADC Demo
Environment: Production

Create Component:
Navigate to Apps -> ADC -> Components, click on "Create Component"
Name: app-service
Display Name: App Service
Click Next
Enter URI: /
Click Next
Workload Group Name: app-wl
URI: http://10.1.1.5:9001
Click "Done"

Click on "Add Backend Workload URI" for http://10.1.1.5:9002 and http://10.1.1.5:9003
Optional: Play with Advanced Settings for load balancing between above three servers.
Click "Submit"

At this stage, you should be able to access the app at http://10.1.1.6/, if load balancing is configured, you'll observer different responses based on configured LB method.

### APIM Use Case ###

# Register a new NGINX instance for API use case
Install Controller Agent:
Infrastructure >> Create Instance >> Add an existing instance
Name: api.nplus.udf
Location: Unspecified
Check - Allow insecure server connections
Copy the CURL script and run on the api.nplus.udf VM



# Create Gateway:

Navigate to Services -> Gateways, click on "+ Gateway" button
Name: api.nplus.udf
Display Name: api.nplus.udf
Environment: Production
Click Next
Instances Refs: api.nplus.udf
Click Next
Hostname: http://10.1.1.7
Click Submit

# Create API Definition
Download OpenAPI Spec file for our sample app : https://raw.githubusercontent.com/lcrilly/ergast-f1-api/master/ergast_oas3.yaml
Navigate to Services -> APIs, click on "Create API Version"
Click on "+ Create New" under API Definition
    - Name: f1-api
    - Display Name: F1 APIs

Click on Browse and select the downloaded Open API Spec yaml file.
Click Next and review the end points.
Click Submit to create the API Definition and Version.

Next step is the map API definition with App Server, this process is called publishing.

Select F1 APIs and click on "+ Add Published API"
    - Name: f1-v1
    - Display Name: F1-v1
    Click Next for Deployment Settings
    - Choose Environment as Production
    - App : API
        - Create a new App with name as "API"
    - Gateway : api.nplus.udf
    Click Next to configure routing
    - We need to create a component which handles these endpoints, click on "Add New" in component tab
     - Name: api-service
     - Display Name: F1 API Service  
     Click Next to configure workload, same process as ADC use case.
     - Name: api-wl
     - URI : http://10.1.1.5:8001 (repeat "Add Backend Workload URI" for http://10.1.1.5:8002)
    Click on Submit to create the component
    - Check "Select All" under Unrouted section to select all endpoints
    - Drag selected endpoints towards the component tab to map it with component.
    Click Submit to Publish API version.

Once the API is publised, use Postman collection to make API calls and ensure that we are receiving response from application servers.

### Developer Portal  ###

# Register a new NGINX instance for API use case
Install Controller Agent:
Infrastructure >> Create Instance >> Add an existing instance
Name: extra.nplus.udf
Location: Unspecified
Check - Allow insecure server connections
Copy the CURL script and run on the extra.nplus.udf VM

# Create Gateway

Navigate to Services -> Gateways, click on "+ Gateway" button
Name: extra.nplus.udf
Display Name: extra.nplus.udf
Environment: Production
Click Next
Instances Refs: extra.nplus.udf
Click Next
Hostname: http://10.1.1.8
Click Submit


# Create Developer Portal
Navigate to Services -> APIs, click on "+ Create Dev Portal" button
- Name: f1-devportal
- Display Name: F1 Developer Portal
- Select Environment : Production
- Select Gateway : extra.nplus.udf
- Select Published APIs: F1-v1
- Custom branding is possible, for this activity, we will just specify Brand Name: F1 Dev Portal
- Click "Submit" to create devportal

Once the Developer Portal is created, you can access it via extra nplus gateway running on http://10.1.1.8
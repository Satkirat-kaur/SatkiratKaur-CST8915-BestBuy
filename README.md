# Lab Project Assignment #2: Building a Cloud-Native App for Best Buy

## Application and Architecture Explanation
### -Application-
This project simulates an online store modeled after Best Buy, showcasing high-quality electronics such as laptops, cameras, PlayStations, and more. Customers can browse products through the store-front interface, where each item displays its name, description, price, and image. Users can add items to their cart and proceed to checkout. Once an order is placed, they’ll receive confirmation on whether the order was successfully submitted.

The application also includes an admin portal (store-admin) for store staff. Through this portal, employees can view a list of customer orders, including product details and total costs. Staff can process and clear orders — once processed, those orders are removed from the admin view.

## Application Architecture
![application architecture diagram](https://github.com/Satkirat-kaur/SatkiratKaur-CST8915-BestBuy/blob/main/Assets/FullStackBestBuy.drawio.png)

### Architecture Overview

This application follows a microservices-based architecture where each service plays a specific role in supporting the customer experience and administrative operations.

---

#### `store-front` (Vue.js)
This is the customer-facing web interface where users browse products and place orders.

- **Communicates with:**
  - `product-service` to fetch product listings.
  - `order-service` to send order details to the order queue.

#### `store-admin` (Vue.js)
This is the admin-facing portal used by employees to manage orders and products.

- **Features:**
  - View and process customer orders.
  - Edit existing products.
  - Create new products using AI.

- **Communicates with:**
  - `makeline-service` for viewing and updating order queue.
  - `product-service` for product management.
  - `ai-service` for generating product descriptions and images.

#### `order-service` (Node.js)
Handles incoming orders from the store-front.

- **Responsibility:**
  - Sends orders to a message queue using either RabbitMQ or Azure Service Bus.

#### `product-service` (Rust)
Manages product data and performs CRUD operations.

- **Used by:**
  - `store-front` to display products.
  - `store-admin` to edit or create new product entries.

- **Note:** Currently works with a static list of sample BestBuy-style products.

#### `makeline-service` (Go)
Serves as a middle layer for order processing and data persistence.

- **Responsibilities:**
  - Retrieves orders from the queue for admin to view.
  - Removes processed orders from the queue.
  - Saves completed orders to the MongoDB database.

#### `ai-service` (Python + FastAPI)
Provides AI-based product generation features.

- **Backed by:**
  - Azure OpenAI (GPT-4 and DALL·E)

- **Used by:**
  - `store-admin` to automatically generate descriptions and images based on product name, price, and keywords.

#### Backing services

- Azure Service Bus/RabbitMQ: this is a backing service. This project wants to switch from RabbitMQ (owned backing service) to Azure Service Bus (managed backing service). It separates the tasks of placing an order (order-service) from processing the order (makeline-service).

- MongoDB: this backing service stores all persistent data related to completed orders
- OpenAI Models: AI models for generating product descriptions (GPT-4) and product images (DALL·E)

## Deployment Instructions (Azure)

### Step 1: Clone the repository
  1. Fork and clone this repository. It includes all the necessary deployment files for this application

### Step 2: Set up the AKS Cluster
  1. Log in to your Azure Portal
  2. Create a resource group in your target region, give it a name
  3. In the portal, search for **Kubernetes services** and click **Create**
  4. Choose your desired configurations
     - Choose the resource group created in the previous step
  5. Use Azure CLI to log into VScode terminal
      - In your newly created Kubernetes service, click on connect > Azure CLI > copy and paste commands
      ```
      az aks get-credentials --resource-group <Resource_Group_Name> --name <ckuster_name>
      ```

### Step 3: Set up the AI Backing Services
  1. Create an Azure OpenAI Service Instance
     - Open Azure Portal
     - Search for Azure OpenAI and click create
     - Choose the region of your choice (I chose Canada Central in my case) for GPT-4 and DALL-E
     - Choose resource group and pricing tier
     - Review + Create
  2. Deploy GPT-4 and DALL-E
     - Go to the created Azure OpenAI resource
     - Go to Model Deployments and click Add Deployment
     - Choose GPT-4 model and deploy it with the wanted configurations
     - Repeat for Dall-e, ensure that they are under the same region for easy use of endpoint
     - Once deployed, go to home, view JSON, and copy the key and the endpoint.
     - Encode the key by doing the following in the terminal (after you have logged in with Azure CLI)
       ```
       echo -n "<your-api-key>" | base64
       ```

  3. In the forked and cloned repository, go to the Deployments folder open secrets.yaml
     - Replace the **OPENAI_API_KEY** placeholder with the Base64-encoded value of the **API_KEY**  
  4. Go the **aps-all-in-one.yaml** file
     - Replace the placeholders with the configurations you retrieved:
     - `AZURE_OPENAI_DEPLOYMENT_NAME`: Enter the deployment name for GPT-4.
     - `AZURE_OPENAI_ENDPOINT`: Enter the endpoint URL for the GPT-4 deployment.
     - `AZURE_OPENAI_DALLE_ENDPOINT`: Enter the endpoint URL for the DALL-E 3 deployment.
     - `AZURE_OPENAI_DALLE_DEPLOYMENT_NAME`: Enter the deployment name for DALL-E 3.
      
### Step 4: Deploy the ConfigMaps and Secrets
- Deploy the ConfigMap for RabbitMQ Plugins:
   ```bash
   kubectl apply -f config-maps.yaml
   ```
- Create and Deploy the Secret for OpenAI API:  
   ```bash
   kubectl apply -f secrets.yaml
   ```
- Verify:
   ```bash
   kubectl get configmaps
   kubectl get secrets
   ```
      
### Step 5: Deploy the Application
   ```bash
   kubectl apply -f aps-all-in-one.yaml
   ```

- Check Pods and Services:
   ```bash
   kubectl get pods
   kubectl get services
   ```
- Once all are ready, go to the **Services and ingresses**, and click on the endpoint for store-front, and store-admin to access the application!



## Table of Microservice Repositories
| **Service**         | **Repository Link**                       |
|---------------------|-------------------------------------------|
| Store-Front         | `https://github.com/Satkirat-kaur/BestBuy-store-front` |
| Store-Admin         | `https://github.com/Satkirat-kaur/BestBuy-store-admin` |
| Order-Service       | `https://github.com/Satkirat-kaur/BestBuy-order-service` |
| Product-Service     | `https://github.com/Satkirat-kaur/BestBuy-product-service` |
| Makeline-Service    | `https://github.com/Satkirat-kaur/BestBuy-makeline-service` |
| Ai-Service          | `https://github.com/Satkirat-kaur/BestBuy-ai-service` |
| BestBuy-virtual-customer          | `https://github.com/Satkirat-kaur/BestBuy-virtual-customer` |


## Table of Docker Images
| **Service**         | **Docker Image Link**                     |
|---------------------|-------------------------------------------|
| store-front-a2         | `https://hub.docker.com/repository/docker/satkiratkaur/store-front/general` |
| store-admin-a2         | `https://hub.docker.com/repository/docker/satkiratkaur/store-admin/general` |
| order-service-a2       | `https://hub.docker.com/repository/docker/satkiratkaur/order-service/general` |
| product-service-a2     | `https://hub.docker.com/repository/docker/satkiratkaur/product-service/general` |
| makeline-service-a2    | `https://hub.docker.com/repository/docker/satkiratkaur/makeline-service/general` |
| ai-service-a2          | `https://hub.docker.com/repository/docker/satkiratkaur/ai-service/general` |


## Any issues or limitations in the implementation

### Issue 1: ai-service did not work 
One major issue I encountered during the lab was with the `ai-service`. While it appeared to be running correctly in the VS Code terminal and Kubernetes pods showed the status as "Running", the service was not functioning as expected in the `store-admin` interface.

I could not access the AI features (description and image generation) from the admin portal. I attempted multiple troubleshooting steps, including:

- Redeploying the `ai-service`
- Deleting and recreating the AI pod and deployment
- Verifying the environment variables and endpoint configurations
- Checking for health check failures

Despite these efforts, the service still did not respond in the UI.

### Issue 2: Azure Quota Limitations

Later, when I tried to redeploy the application on a new cluster for additional testing, I encountered another issue — Azure blocked me from creating a new AKS cluster due to a **quota limit**. This prevented me from spinning up additional resources, which limited my ability to retry the full end-to-end deployment.

This highlights a potential limitation when using Azure on a free or student subscription, where quota restrictions can impact testing and development flexibility.


### Issue 3: RabbitMQ Replacement with Azure Service Bus

**Problem**:  
Originally, RabbitMQ was configured and working successfully. However, when I recreated the resource later for my demo video, RabbitMQ stopped functioning as expected. In an effort to continue, I attempted to replace RabbitMQ with Azure Service Bus for message queuing.

Since I was unable to use Managed Identity authentication, I opted for the Shared Access Signature (SAS) policy method. Although the store-front was still able to send orders to the queue, the store-admin interface could no longer retrieve them. The application failed to fully process orders, and errors started appearing in the console.

**What I Tried**:
- Updated the environment variables in the `aps-all-in-one.yaml` file to use Azure Service Bus credentials.
- Created a new `secret.yaml` to store the Service Bus password.
- Later switched to using the **Primary Connection String**
- Removed the RabbitMQ deployment section entirely to prevent conflicts.
- Modified both `order-service` and `makeline-service` configurations to reflect the Azure Service Bus setup.
- Inspected Azure Service Bus metrics using **Service Bus Explorer**.
- Attempted debugging by checking and adding log statements in the service code, but it was unclear where logs were showing up.

**Final Outcome**:  
After hours of testing, editing, and troubleshooting, Azure Service Bus still did not work with my application. Because the store-admin service could not process incoming orders. 
This experience highlighted the complexity of switching message brokers mid-project and the importance of maintaining consistent resource configuration across environments.

---

## Demo Video
[Demo Video Youtube Link] (https://www.youtube.com/watch?v=WS-JH0ePkbE)








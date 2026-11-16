# Empire Simulation Map
A cloud-native strategic simulator for modeling territorial expansion, resource management, and terrain-based pathfinding.

### About
Simulating the expansion of an empire across a vast map requires intense computational power. Calculating terrain penalties, the cost of occupying specific provinces, and the logistics of maintaining settlements can quickly overload a standard client-side application, especially as the number of controlled provinces grows.

The Empire Simulation Map solves this by offloading the heavy lifting to the cloud. Users can log in, load their saved empire states, plan their territorial expansions, and submit their turn. Behind the scenes, the complex pathfinding and cost algorithms are decoupled via a message queue and processed asynchronously by serverless functions. 

The entire backend infrastructure is hosted on Microsoft Azure. All sensitive data—like user profiles and game saves—is stored securely in a private database. The system uses managed identities for internal authentication, runs inside a secure virtual network, and the infrastructure is fully scripted for automated deployment and teardown to optimize credit usage.

### Features
- Sign-in with a Microsoft account (Azure Entra ID)
- Interactive map interface where users can select provinces to occupy or build settlements
- Asynchronous turn processing for high-complexity terrain calculations
- Save/Load functionality to persist empire states and historical scenarios in the cloud
- In-app status updates notifying the user when their expansion calculations are complete

### Architecture
The frontend web app (built with React and WebAssembly) communicates with an API hosted on Azure App Service. When a user submits an expansion plan, the API validates the request and places an "expansion job" message into an Azure Service Bus queue. 

An Azure Function triggers on this message, executing the heavy terrain penalty calculations and resource deductions. Once the math is done, the Function updates the user's empire state in an Azure SQL Database and saves the heavy JSON map snapshots into Azure Blob Storage. Shared across all components: VNet + NSG, Private Endpoints (SQL, Blob), Entra ID, Managed Identity, Key Vault, and Application Insights.

### Azure Services
| Area | Services |
| :--- | :--- |
| **Compute** | App Service (API backend), Azure Functions (Service Bus trigger for heavy calculations), Azure Static Web Apps (Frontend) |
| **Storage** | Blob Storage (Map state JSONs, large assets), Azure SQL Database (User profiles, save slots, resources) |
| **Messaging** | Service Bus (Decoupling expansion calculation requests) |
| **Networking**| Virtual Network, NSG, Private Endpoints, Private DNS |
| **Security** | Entra ID (User login), Managed Identity, Key Vault |
| **Observability** | Application Insights, Azure Monitor |
| **Deployment**| Terraform / Azure CLI, GitHub Actions |

### Example flow
1. The user signs in via Microsoft Entra ID and loads their current empire save from the database.
2. The user selects a direction in which to expand their empire, queues the actions, and clicks "Process Turn".
3. The API receives the command and drops a message into the Service Bus queue so the UI doesn't freeze.
4. The background Azure Function picks up the message, calculates the high terrain penalties and movement costs, deducts the resources, and saves the new map state to the SQL Database and Blob Storage.
5. The frontend polls or receives a notification that the turn is ready, refreshing the map to display the newly conquered territories and updated resource pools.
